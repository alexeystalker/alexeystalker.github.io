---
share: true
tags:
 - NET/SignalR
 - Redis
---
# Используем множество реплик хаба с помощью объединительной панели Redis
Сперва добавим нужный NuGet пакет:
```bash
dotnet add package Microsoft.AspNetCore.SignalR.StackExchangeRedis
```
Затем, добавим инициализацию к методу `AddSignalR()`:
```csharp
builder.Services.AddSignalR().AddStackExchangeRedis("тут строка подключения Redis");
```
Можно использовать обычный формат строки подключения. Например, для Redis, запущенного на локальной машине со стандартным портом, это будет `localhost:6379` или `127.0.0.1:6379`.

Теперь, для того, чтобы запустить несколько экземпляров сервера SignalRServer на нашей машине, нам нужно скопировать содержимое папки с скомпилированным проектом (обычно папка bin), в другое место на машине, а также поменять порты в файле **launchSettings.json**.

---
Однако, можно сделать по другому. Мы вынесем код нашего хаба в библиотеку классов (class library), и добавим её как в наш изначальный проект (SignalRServer), так и в другой.
## Перенесем хаб в библиотеку
добавим новый проект в решение:
```bash
dotnet new classlib -o SignalRHubs
dotnet sln add SignalRHubs/SignalRHubs.csproj
```
И добавим нужные пакеты к проекту SignalRHubs:
```bash
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
dotnet add package Microsoft.AspNetCore.SignalR.StackExchangeRedis
```
После этого их можно удалить из проекта SignalRServer.csproj
Далее, перенесем файлы **LearningHub.cs** и **ILearningHubClient.cs** в проект SignalRHubs и сменим в них пространство имен (namespace) на `SignalRHubs`.
Теперь, чтобы наш код компилировался, подключим проект SignalRHubs к проекту SignalRServer, а также добавив `using SignalRHubs;` в файле **Program.cs**.
Всё, теперь наш код компилируется.
## Добавим другое приложение
Можно встроить SignalR в приложение ASP.NET Core любого типа. Для примера, используем для встраивания ещё одно MVC приложение.
Для этого нам нужно создать новый проект и добавить его к решению
```bash
dotnet new mvc -o SignalRServer2
dotnet sln add SignalRServer2/SignalRServer2.csproj
```
Далее, подключим проект SignalRHubs и добавим SignalR к нашим сервисам, а также добавим конечную точку - хаб
```csharp
using SignalRHubs; //не забыть юзинг
...
builder.Services.AddSignalR().AddStackExchangeRedis("Строка подключения Redis, такая же, как в первом сервере");
...
app.MapHub<LearningHub>("/learningHub");
```
Далее нужно либо скопировать код, инициализирующий аутентификацию и авторизацию из SignalRServer в SignalRServer2, либо удалить его из SignalRServer, чтобы наличие авторизационной логики совпадало у обоих приложений. Для демонстрационных целей подойдут оба пути.
Также не забудем назначить другой порт в **launchSettings.json**.
## Запустим наш распределенный хаб
Теперь, после того, как мы запустим оба наших сервера и подключим клиентов, указав адреса либо первого, либо второго серверов, и отправим сообщения для всех клиентов, то увидим, что сообщение получили все клиенты, независимо от того, к какому из серверов они подключены.