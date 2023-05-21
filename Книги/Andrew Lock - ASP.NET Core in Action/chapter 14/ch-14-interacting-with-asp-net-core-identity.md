---
share: true
tags:
 - NET/ASPNETCore/Identity
 - security
---
# Взаимодействие с ASP.NET Core Identity
Точкой входа по умолчанию является страница регистрации пользователя. На ней создается новая сущность `IdentityUser` с адресом электронной почты и паролем, после чего пользователь перенаправляется на страницу подтверждения адреса почты. Однако по умолчанию сервис электронной почты не активирован[^1]. По умолчанию адреса должны быть уникальными, а пароль — удовлетворять определенным требованиям. Эти и другие параметры можно настроить в конфигурационной лямбда-функции метода `AddDefaultIdentity()`:
```csharp
services.AddDefaultIdentity<IdentityUser>(options =>
{
	options.SignIn.RequireConfirmedAccount = true; //требуется подтверждение email
	options.Lockout.AllowedForNewUsers = true; //блокировка, чтоб предотвратить атаки методом перебора
	options.Password.RequiredLength = 12;
	options.Password.RequireNonAlphanumeric = false;
	options.Password.RequireDigit = false;
})
```

После регистрации пользователю необходимо выполнить вход. Шаблон UI по умолчанию содержит описание добавления сторонних поставщиков входа в учётную запись. Это одна из причин, по которой может потребоваться [[ch-14-customizing-a-page-in-identity-default-ui|настроить]] шаблоны по умолчанию.[^2]

После входа пользователь получает доступ к страницам управления UI Identity. Они позволяют менять адрес почты, пароль, настраивать двухстороннюю аутентификацию при помощи приложения-аутентификатора.[^3]

[^1]: Как активировать отправку электронной почты, см. [здесь](https://docs.microsoft.com/en-us/aspnet/core/security/authentication/accconfirm?view=aspnetcore-5.0&tabs=visual-studio)
[^2]: альтернатива - полностью отключить регистрацию пользователей, см. [здесь](https://docs.microsoft.com/en-us/aspnet/core/security/authentication/scaffold-identity?view=aspnetcore-6.0&tabs=visual-studio#disable-register-page)
[^3]: добавляем генератор QR-кода для страницы активации двухфакторной аутентификации: [документация](https://docs.microsoft.com/en-us/aspnet/core/security/authentication/identity-enable-qrcodes?view=aspnetcore-6.0)