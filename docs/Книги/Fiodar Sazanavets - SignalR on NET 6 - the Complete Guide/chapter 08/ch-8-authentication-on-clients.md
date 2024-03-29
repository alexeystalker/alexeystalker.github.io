---
share: true
tags:
 - NET/SignalR
 - security
 - NET/authentication
---
# Аутентификация на клиентах
## Аутентификация в клиенте JavaScript
Так как мы используем аутентификацию по cookie, в javaScript клиенте делать ничего не нужно — сервер сконфигурирован перенаправлять пользователя на страницу поставщика SSO для логина, после чего cookie с аутентификационной информацией будет добавлена к запросу.
Однако, если JavaScript-клиент используется в другом контексте, нам может понадобиться использовать для аутентификации токен вместо cookie. Для этого к опциям, передаваемым в метод `withUrl`[^1], добавляется строчка:
```js
accessTokenFactory: () => myToken
```
где `myToken` — токен, получаемый от поставщика SSO[^2].
Также у клиента JavaScript есть опция `withCredentials`, которая, будучи установленной в `true`[^3], позволяет передавать аутентификационную информацию из кросс-доменных cookie в запрос. Это особенно важно при использовании Azure App Service, так как этот сервис (как и многие другие) использует cookie для липкости, и не будет правильно работать без задания этой опции.
## Аутентификация в клиенте .NET
В файле **DotnetClient/Program.cs** добавим (после ввода URL хаба) следующее:
```csharp
Console.WriteLine("Please specify the access token");
var token = Console.Readline();
```
И затем изменим вызов метода `withUrl`:
```csharp
.WithUrl(url, options => {
	options.AccessTokenProvider = () => Task.FromResult(token);
})
```
Здесь мы просто передадим токен, который мы скопипастим из консоли сервера. Но в реальном приложении делегат, переданный в `AccessTokenProvider`, должен получить токен от провайдера SSO.
Кроме использования `AccessTokenProvider` у нас есть и другие возможности для аутентификации. Мы можем использовать свойство `Cookies`, хотя этот способ не является рекомендуемым для такого рода клиентов. Также у нас есть свойство `Credentials`, позволяющее отправить серверу аутентификационную информацию (логин-пароль) вместо токена. И, наконец, у нас есть свойство `UseDefaultCredentials`, позволяющее передать аутентификационную информацию Microsoft Active Directory (доступно только под Windows).
## Аутентификация клиента Java
Для использования токена в [[ch-7-configuring-java-client|клиенте Java]] нужно добавить вызов функции `withAccessTokenProvider`, передав в аргументе функцию, возвращающую строку с токеном.
## Получение токена от поставщика SSO
Сперва необходимо запустить приложение AuthProvider. Затем запускаем приложение SignalRServer. При заходе на домашнюю страницу приложения SignalRServer, мы будем перенаправлены на страницу логина, где надо ввести логин и пароль одного из пользователей, созданых [[ch-8-configuring-sso-application|на этом этапе]]. После успешной аутентификации мы будем перенаправлены обратно на домашнюю страницу, при этом аутентификационный токен в формате [[json-web-token|JWT]] будет выведен в консоль приложения SingalRServer.


[^1]:Подробнее о опциях клиента JavaScript смотрите [[ch-7-configuring-javascript-client|тут]];
[^2]:Мы не будем рассматривать процесс получения токена из JavaScript. Об этом можно прочитать в [документации OpenID Connect](https://openid.net/connect/);
[^3]:Согласно [документации](https://learn.microsoft.com/en-us/javascript/api/@microsoft/signalr/ihttpconnectionoptions?view=signalr-js-latest#@microsoft-signalr-ihttpconnectionoptions-withcredentials), это значение по умолчанию

