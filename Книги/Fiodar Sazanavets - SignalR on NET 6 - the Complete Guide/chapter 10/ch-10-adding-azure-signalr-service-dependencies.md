---
share: true
tags:
 - NET/SignalR
 - Azure
---
# Добавляем зависимости Azure SignalR Services
Сперва добавляем NuGet-пакет к проекту SignalRHubs:
```bash
dotnet add package Microsoft.Azure.SignalR
```
Затем добавляем сервисы в наш проект SignalRServer:
```csharp
builder.Services.AddSignalR().AddAzureSignalR();
```
Тоже самое можно сделать и в проекте SignalRServer2.
Есть два способа правильно сконфигурировать сервис. Первый - вызвать `AddAzureSignalR()` со строковым параметром - строкой подключения. Второй - вызвать метод без параметра. В этом случае, для того, чтобы всё заработало, нужно воспользоваться инструментом User Secrets[^1] внутри проекта.
Предположим, мы не стали передавать строку подключения в параметре. Тогда нам нужно выполнить в папке проекта команды
```bash
dotnet user-secrets init
dotnet user-secrets set Azure:SignalR:ConnectionString "%Строка подключения Azure%"
```
`Azure:SignalR:ConnectionString` - это ключ, по которому промежуточное ПО будет искать строку подключения, если таковой не будет указано явно (параметром).

На этом процесс добавления Azure SignalR Services в проект завершён. Теперь при подключении клиент будет перенаправлен в сервис. При этом клиенты всё ещё могут вызывать методы хаба, однако все команды отправки сообщений будут перенаправляться в облако.

Остаётся вопрос с использованием `IHubContext`, так как реализация по умолчанию несовместима с Azure SignalR Service. Разберемся, как это можно починить.
## Используем HubContext в связке с Azure SignalR Service
Как вы помните из [[ch-9-using-hubcontext-to-send-messages-from-outside-signalr-hub|предыдущей главы]], HubContext - это механизм для отправки сообщений извне хаба SignalR.
Когда мы используем “монолитный” хаб SignalR или масштабируем его при помощи объединительной панели Redis, реализация интерфейса `IHubContext` автоматически внедряется в конструкторе вызывающего класса. Это обеспечивается внутри метода `AddSignalR()`. В случае с Azure SignalR Service это не сработает.
Существует специальная библиотека, позволяющая внедрять особую реализацию `IHubContext`, совместимую с Azure SignalR Service. Вот как она применяется.
Сперва добавим NuGet-пакет в проект SignalRHubs:
```bash
dotnet add package Microsoft.Azure.SignalR.Management
```
Далее добавим в файл **Program.cs** проекта SignalRServer:
```csharp
using Microsoft.Azure.SignalR.Management;
//... тут прочие юзинги и начало файла...
var serviceManager = new ServiceManagerBuilder()
	.WithOptions(option => 
	{
		option.ConnectionString = "%строка подключения Azure%"ж
	})
	.BuildServiceManager();

var hubContext = await serviceManager.CreateHubContextAsync<LearningHub>("LearningHub", CancellationToken.None);
builder.Services.Add(new ServiceDescriptor(typeof(IHubContext<LearningHub>), hubContext));
//... где-то задесь вызов builder.Build()
```
Здесь мы создаем специальную реализацию `IHubContext` и переопределяем реализацию по умолчанию, зарегистрированную ранее, поэтому важно, чтобы этот код стоял как можно ближе к вызову `builder.Build()`.

[^1]: О инструменте User Secrets, он же *менеджер пользовательских секретов* также упоминает Э. Лок [[ch-11-storing-config-secrets-safely#Хранение секретов с помощью менеджера пользовательских секретов в окружении разработки|тут]]