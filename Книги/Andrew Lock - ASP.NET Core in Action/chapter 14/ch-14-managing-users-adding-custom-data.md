---
share: true
tags:
 - NET/ASPNETCore/Identity
---
# Управление пользователями: добавление специальных данных для пользователей
Настроим объект `ClaimsPrincipal`, добавляя дополнительные утверждения в таблицу AspNetUserClaims при создании пользователя, также разберем, как получить доступ к этим объектам на страницах Razor и шаблонах.

Есть два случая, когда пользователю нужно добавить утверждения:
- *каждому пользователю при регистрации*;
- *вручную, после того, как пользователь зарегистрировался*.

> [!tip] Совет
> Еще одним распространенным подходом является настройка сущности `IdentityUser`. Этот подход описан в [статье](https://docs.microsoft.com/en-us/aspnet/core/security/authentication/add-user-data?view=aspnetcore-6.0&tabs=visual-studio).

Предположим, мы хотим добавить новое утверждение `FullName`. Типичный подход будет выглядеть так:
1. [[ch-14-customizing-a-page-in-identity-default-ui|Добавляем]] заготовки для Register.cshtml;
2. Добавляем поле "Имя" в `InputModel` в `PageModel` файла Register.cshtml.cs;
3. Добавляем поле ввода "Имя" в шаблон представления Razor Register.cshtml;
4. Создаем новую сущность `ApplicationUser` в обработчике `OnPost()` путём вызова метода `CreateAsync` для `UserManager<ApplicationUser>`;
5. Добавляем пользователю новое утверждение при помощи `UserManager.AddClaimAsync()`;
6. Продолжаем метод, как и раньше, отправив электронное письмо или выполнив вход, если письмо не требуется.

Шаги 4-6 рассмотрим в следующем листинге; сосредоточимся на строках, добавляющих дополнительное утверждение.
```csharp
public async Task<IActionResult> OnPostAsync(string returnUrl = null)
{
	if( ModelState.IsValid)
	{
		var user = new ApplicationUser 
		{
			UserName = Input.Email,
			Email = Input.Email
		};
		var result = await _userManager.CreateAsync(user, Input.Password);
		if (result.Succeeded)
		{
			var claim = new Claim("FullName", Input.Name);
			await _userManager.AddClaimAsync(user, claim);
			
			var code = await _userManager.GenerateEmailConfirmationTokenAsync(user);
			await _emailSender.SendEmailAsync(Input.Email, "Confirm your email", code);
			await _signInManager.SignInAsync(user);
			return LocalRedirect(returnUrl);
		}
		foreach (var error in result.Errors)
		{
			ModelState.AddModelError(string.Empty, error.Description);
		}
	}
	return Page();
}
```

При вызове `_signInManager.SignInAsync(user)` принципал `ClaimsPrincipal` будет задан свойству `HttpContext.User`, и его утверждения можно получить везде, где к нему есть доступ. Например в шаблоне Razor:
```csharp
@User.Claims.FirstOrDefault(x => x.Type == "FullName")?.Value
```
Identity включает в себя удобный метод расширения для подобной задачи:
```csharp
@User.FindFirstValue("FullName");
```