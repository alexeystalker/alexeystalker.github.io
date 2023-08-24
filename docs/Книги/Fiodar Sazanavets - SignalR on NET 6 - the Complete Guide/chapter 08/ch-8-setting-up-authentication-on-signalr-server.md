---
share: true
tags:
 - NET/SignalR
 - security/OIDC
 - NET/authentication
---
# Настраиваем аутентификацию на сервере
Сперва нам нужно добавить NuGet пакеты JwtBearer и OpenIdConnect к нашему проекту SignalRServer:
```bash
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
dotnet add package Microsoft.AspNetCore.Authentication.OpenIdConnect
```
Далее добавим нужные юзинги в файл **Program.cs**:
```csharp
using Microsoft.AspNetCore.Authentication.Cookies;
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;
using System.IdentityModel.Tokens.Jwt;
```
Теперь нам надо сконфигурировать промежуточное ПО. Добавим следующий код перед вызовом `builder.Build()`:
```csharp
builder.Services.AddAuthentication(options =>
{
	options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
	options.DefaultChallengeScheme = "oidc";
})
.AddCookie(CookieAuthenticationDefaults.AuthenticationScheme)
.AddOpenIdConnect("oidc", options =>
{
	options.Authority = "https://localhost:5001";
	options.ClientId = "webAppClient";
	options.ClientSecret = "webAppClientSecret";
	options.ResponseType = "code";
	options.CallbackPath = "/signin-oidc";
	options.SaveTokens = true;
	options.RequireHttpsMetadata = false;
})
```
Здесь мы сперва задаём схему аутентификации и схему запроса (challenge), показывая, что мы хотим использовать OpenIdConnect (`"oidc"`) в качестве механизма аутентификации. Затем задаем обработчик аутентификации при помощи куки (`AddCookie()`), указав ту же схему аутентификации, что и на первом этапе.
После этого мы конфигурируем OpenIdConnect для работы с [[ch-8-setting-up-single-sign-on-provider|нашим поставщиком SSO]]. Обычно все эти настройки хранятся в конфигурационных файлах и различаются для разных окружений. Но для примера мы захардкодим их.
- `Authority` — базовый URL поставщика SSO;
- `ClientId` — Id клиента, который мы указали при [[ch-8-configuring-sso-application|конфигурировании поставщика]];
- `ClientSecret` — секрет клиента, указанный там же. Обычно берется из шифрованного источника, такого как Azure Key Vault или [[hashicorp-vault|Hashicorp Vault]];
- `ResponseType` — OpenId Connect может отвечать разными способами. Тип ответа `code` означает ответ в виде зависящего от времени кода, который возвращается клиенту в случае успешной аутентификации;
- `CallbackPath` — путь для редиректа в случае успешной аутентификации. Базовым URL будет URL этого приложения;
- `SaveTokens` — если эта опция включена, приложение будет сохранять токен аутентификации в cookie. Там же будут храниться токены обновления (refresh tokens).
- `RequireHttpsMetadata` — эту опцию необходимо включить, если провайдер SSO доступен по нешифрованному протоколу HTTP. Так как у нас используется HTTPS, в этой опции нет нужды.

Мы добавили cookie-аутентификацию для браузерных клиентов. Но что делать клиентам, у которых нет возможности перейти на страницу аутентификации? Для них добавим дополнительный обработчик, который может обрабатывать токен [[json-web-token|JWT]]. Для этого дополним предыдущий вызов следующим:
```csharp
.AddJwtBearer(options =>
{
	options.Authority = "https://localhost:5001";
	options.TokenValidationParameters = new TokenValidationParameters
	{
		ValidateAudience = false
	};
	options.RequireHttpsMetadata = false;
	options.Events = new JwtBearerEvents
	{
		OnMessageReceived = context =>
		{
			var path = context.HttpContext.Request.Path;
			if(path.StartsWithSegments("/learningHub"))
			{
				// Пробуем достать токен из строки запроса
				var accessToken = context.Request.Query["access_token"];
				//Если нет, достаём из заголовка Authorization
				if(string.IsNullOrWhiteSpace(accessToken))
				{
					sccessToken = context.Request.Headers["Authorization"]
						.ToString()
						.Replace("Bearer ", "");
				}
				context.Token = accessToken;
			}
			return Task.CompletedTask;
		}
	};
});
```
Здесь мы отключили валидацию специального клейма **audience** в токене в настройке `TokenValidationParameters`. Этот клейм указывает, обращение к каким эндпоинтам может быть аутентифицировано этим токеном.
Также в обработчике `OnMessageReceived` мы очищаем токен от указания на его тип (это указание появляется в случае, когда токен достаётся из заголовка `Authorization`).
Некоторые типы запросов, например, инициированнное браузером Websocket-подключение, или SSE не могут использовать заголовки, в том числе Authorization. В этом случае токен достаётся из строки запроса (параметр `access_token`).
Отметим, что по умолчанию обработчик JWT применяет токен “как есть”, то есть все клеймы, указанные в токене, будут перенесены в приложение без изменений. С другой стороны, обработчик аутентификации с помощью cookie, а также обработчик OIDC имеют свою логику маппинга клеймов, и клеймы из токена могут быть сопоставлены клеймам с другими именами. Это поведение управляется настройкой `JwtSecurityTokenHandler.DefaultMapInboundClaims`. К примеру, эта команда запрещает сопоставлять клеймы токена во встроенные клеймы:
```csharp
JwtSecurityTokenHandler.DefaultMapInboundClames = false;
```
А вот эта команда позволяет очистить уже сопоставленные клеймы:
```csharp
JwtSecurityTokenHandler.DefaultMapInboundClaims.Clear();
```
Теперь, когда наше промежуточное ПО настроено, необходимо его включить в конвейер промежуточного ПО:
```csharp
app.UseAuthentication();
app.UseAuthorization();
```
Теперь ограничим доступ к нашему хабу SignalR. Для этого добавим юзинги в файл **LearningHub.cs** (или же можно пометить их как `global` в другом месте проекта):
```csharp
using Microsoft.AspNetCore.Authentication.Cookies;
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.AspNetCore.Authorization;
```
Теперь добавим атрибут `[Authorize]` к заголовку класса:
```csharp
[Authorize(AuthenticationSchemes = JwtBearerDefaults.AuthenticationScheme + "," +
		  CookieAuthenticationDefaults.AuthenticationScheme)]
```
Таким образом, мы разрешаем доступ к методам хаба только аутентифицированным пользователям. И это ровно то, что мы делаем в любом другом ASP.NET Core приложении[^1]. Нам пришлось явно указывать используемые схемы аутентификации, так как в противном случае использовалась бы только схема, указанная в качестве схемы по умолчанию.

Нам осталось сделать две вещи. Во-первых, нам нужно вывести `access_token` в логи сервера, чтобы мы могли скопировать его и вставить в заголовки запросов от клиентов, и во-вторых - добавить возможность логаута (logout). Всё это мы сделаем внутри файла **HomeController.cs**.
Сперва, как обычно, юзинги:
```csharp
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.Cookies;
using Microsoft.AspNetCore.Authorization;
```
Далее, добавим отправку токена на консоль в начале метода `Index`:
```csharp
var accessToken = await HttpContext.GetTokenAsync("access_token");
Console.WriteLine($"Access token: {accessToken}");
```
Наконец, добавим метод `Logout`:
```csharp
public IActionResult LogOut()
{
return new SignOutResult(new[] {CookieAuthenticationDefaults.AuthenticationScheme, "oidc"});
}
```
Чтобы иметь доступ к этому методу с web-страниц, откроем файл **\_Layout.cshtml** и добавим такую разметку внутрь элемента `ul` с классом `nav-bar`:
```html
<li class="nav-item">
	<a class="nav-link text-dark" asp-area="" asp-controller="Home" asp-action="LogOut">Log Out</a>
</li>
```

[^1]: О авторизации в приложениях ASP.NET Core подробно написано у Эндрю Лока [[ch-15-authorization|здесь]].