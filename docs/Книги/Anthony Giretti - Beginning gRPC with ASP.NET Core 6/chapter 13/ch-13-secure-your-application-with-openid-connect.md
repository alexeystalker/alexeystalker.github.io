---
share: true
tags:
 - security/OIDC
---
# Обезопасьте своё приложение с помощью OpenId Connect
#### Введение в OpenId Connect
> [!Note] От меня
> Так как [[oidc|OpenId Connect]] стал фактически стандартом для процесса аутентификации и упоминается во многих учебниках, я перенес текст этого раздела в отдельную заметку.

#### [[ch-13-configure-asp-net-core|Конфигурация ASP.NET Core]]
#### [[ch-13-use-grpcurl-and-grpcgui-with-a-jwt|Используем gRPCurl и gRPCui с JWT]]
#### [[ch-13-use-a-c-sharp-client-with-a-jwt|Используем клиент на C# с JWT]]
#### [[ch-13-use-a-grpc-web-client-with-a-jwt|Используем клиент gRPC-web с JWT]]
#### Получаем данные о пользователе на стороне сервера
Чтобы получить данные о пользователе, нужно, как это обычно и делается, [[ch-14-users-and-claims-in-asp-net-core|достать]] [[aspnetcore-principal|принципала]] из HTTP-контекста, который можно получить из gRPC-контекста:
```csharp
var user = context.GetHttpContext().User;
```