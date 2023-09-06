---
share: true
tags:
 - security/OIDC
---
# Конфигурация ASP.NET Core
> [!Note] От меня
> В данном разделе рассмотрен authentication flow с использованием схемы Bearer Token, но это не единственная возможная схема.

Первым шагом добавим в наше приложение необходимый NuGet-пакет:
```powershell
Install-Package Microsoft.AspNetCore.Authentication.JwtBearer
```
Далее сконфигурируем [[authentication|аутентификацию]] и [[authorization|авторизацию]] в **Program.cs**:
```csharp
...
builder.Services.AddAuthentication(options =>
{
	options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
	options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
}).AddJwtBearer(options =>
{
	// конкретные значения параметров зависят от выбранного сервиса аутентификации!
	options.Authority = "..."; //Тут адрес сервера аутентификации
	options.Audience = "..."; //Целевое приложение (иногда требуется некоторыми серверами)
	options.TokenValidationParameters.ValidateLifetime = true;
	options.TokenValidationParameters.ValidateIssuer = true;
	options.TokenValidationParameters.ClockSkew = TimeSpan.FromMinutes(5);
});
builder.Services.AddAuthorization();

//...

app.UseAuthentication();
app.UseAuthorization();

//...

```
Здесь мы добавили следующее:
- метод расширения `AddAuthentication()` с указанием схемы аутентификации JWT Bearer;
- метод расширения `AddJwtBearer()`, который позволяет указать такие параметры как `Authority` (адрес сервера аутентификации, который валидирует токен), `Audience` (указывается id приложения, для которого выпущен токен) — обычно значения этих параметров предоставляются используемым сервисом аутентификации. Также можно сконфигурировать параметры валидации токена;
- метод расширения `AddAuthorization()` — добавляет возможность настройки авторизации при помощи атрибута `Authorize`;
- `UseAuthentication()` добавляет миддлварю аутентификации в конвейер[^1];
- `UseAuthorization()` добавляет миддлварю авторизации в конвейер[^2].

> [!attention] Внимание!
> Обе миддлвари должны быть добавлены в конвейер *после* `app.UseCors()` и `app.UseGrpcWeb()`

[^1]: Об аутентификации подробно [[ch-14-authentication|здесь]]
[^2]: Об авторизации подробно [[ch-15-authorization|здесь]]

Теперь можно декорировать весь класс сервиса Grpc или отдельные методы атрибутом `Authorize` (если вы декорировали весь класс, но хотите, чтобы какой-то метод остался доступен неаутентифицированным пользователям, используйте атрибут `allowAnonymous`). Также работают все остальные возможности авторизации, например роли или [[claim|клеймы]]. При этом, если к закрытому авторизацией методу обратится неаутентифицированный пользователь, вернётся ошибка с [[ch-3-grpc-status|gRPC-статусом]] `UNAUTHENTICATED`, а если это будет аутентифицированный, но не имеющий права доступа к методу пользователь — gRPC-статус будет `PERMISSIONDENIED`. При этом ASP.NET Core отступает от спецификации gRPC и возвращает HTTP-статус `401 Unauthorized` в первом случае и `403 Forbidden` во втором (тогда как согласно спецификации HTTP-статус должен быть `200 OK` в любом случае). 