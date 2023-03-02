---
share: true
tags:
 - NET/SignalR
---
# Используем HubContext для отправки сообщений извне хаба SignalR
SignalR поставляется с интерфейсом `IHubContext`, реализации которого дают доступ к всем полям хаба SignalR, включая группы, клиентов и так далее. Для использования интерфейса нужно внедрить его в конструкторе нужного класса. Все необходимые зависимости регистрируются, когда вызывается метод `builder.Services.AddSignalR()`. `HubContext` можно использовать не только в случае распределённых хабов, но и в случае “монолитных”, однако он особенно к месту именно в случае распределённого хаба, поэтому мы обсуждаем его именно в этой главе.
## Использование HubContext в приложении
Откроем файл **HomeController.cs** проекта SignalRServer и добавим следующие юзинги:
```csharp
using Microsoft.AspNetCore.SignalR;
using SignalRHubs;
```
Теперь внедрим интерфейс:
```csharp
private readonly IHubContext<LearningHub> hubContext;
public HomeController(IHubContext<LearningHub> hubContext)
{
	this.hubContext = hubContext;
}
```
Или, если мы хотим использовать [[ch-2-making-hub-strongly-typed|типизированный]] контекст:
```csharp
private readonly IHubContext<LearningHub, ILearningClient> hubContext;
public HomeController(IHubContext<LearningHub, ILearningClient> hubContext)
{
	this.hubContext = hubContext;
}
```
Теперь добавим к действию `Index` такой код:
```csharp
//В случае нетипизированного контекста
await hubContext.Clients.All.SendAsync("ReceiveMessage", "Index page has been opened by a client");
//В случае типизированного контекста
await hubContext.Clients.All.ReceiveMessage("Index page has been opened by a client");
```
## Тестируем HubContext на распределенном хабе SignalR
Теперь запустим приложения SignalRServer и SignalRServer2, затем запустим DotnetClient и подключим его к SignalRServer2. Теперь, если обновить домашнюю страницу приложения SignalRServer, DotnetClient получит сообщение `Index page has been opened by a client` будучи подключённым к другому экземпляру хаба.

> [!info] От меня
> Существует ещё один момент, не отражённый автором. Дело в том, что классы-хабы регистрируются в [[di-container|контейнере зависимостей]] с временем жизни Transient, а экземпляры `IHubContext` имеют время жизни Singleton. В том числе поэтому и не стоит добавлять хаб в качестве зависимости к сторонним классам, а также размещать в хабе логику, которая может быть вызвана из другого класса. Для отправки сообщений нужно использовать `IHubContext`, как показано выше.