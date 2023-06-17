---
share: true
tags:
 - NET/SignalR
 - SSO
 - security
 - NET/templates
---
# Настраиваем IdentityServer 4
Откроем терминал в в папке решения `LearningSignalR`. Сперва скачаем [[project-template|шаблоны проектов]] IdentityServer4:
```bash
dotnet new -i IdentityServer4.Templates
```
Далее создадим проект из шаблона[^1], содержащего Admin UI — в этом шаблоне уже есть user-friedndly интерфейс:

```bash
dotnet new is4admin -o AuthProvider
```
Теперь добавим проект к решению:
```bash
dotnet sln add AuthProvider/AuthProvider.csproj
```
Нам нужно добавить немного кода, чтобы вся необходимая информация попадала в аутентификационный токен. Создадим файл **UserProfileService.cs** в проекте AuthProvider со следующим содержимым (юзинги я опущу):
```csharp
namespace AuthProvider
{
	public class UserProfileService: IProfileService
	{
		private readonly IUserClaimsPrincipalFactory<ApplicationUser> claimsFactory;
		private readonly UserManager<ApplicationUser> usersManager;

		public UserProfileService(
			UserManager<ApplicationUser> usersManager,
			IUserClaimsPrincipalFactory<ApplicationUser> claimsFactory)
		{
			this.usersManager = usersManager;
			this.claimsFactory = claimsFactory;
		}
	}
}
```
Наш класс реализует интерфейс `IProfileService` из неймспейса `IdentityServer4.Services`, поэтому нам нужно добавить реализации методов этого интерфейса. Сперва реализуем метод `GetProfileDataAsync`:
```csharp
public async Task GetProfileDataAsync(ProfileDataRequestContext context)
{
	var subject = context.Subject.GetSubjectId();
	var user = await usersManager.FindByIdAsync(subject);
	var claimsPrincipal = await claimsFactory.CreateAsync(user);
	var claimsList = claimsPrincipal.Claims.ToList();

	claimsList = claimsList.Where(c => context.RequestedClaimTypes.Contains(c.Type)).ToList();

	//Добавим клеймы, специфичные для пользователя
	claimsList.Add(new Claim(JwtClaimTypes.Email, user.Email));
	claimsList.Add(new Claim(JwtClaimTypes.Name, user.UserName));

	if(usersManager.SupportsUserRole)
	{
		foreach(var roleName in await usersManager.GetRolesAsync(user))
		{
			claimsList.Add(new Claim(JwtClaimTypes.Role, roleName));
			//Специальный клейм для админа
			if(roleName == "admin")
				claimsList.Add(new Claim("admin", "true"));
		}
	}
	context.IssuedClaims = claimsList;
}
```
Токен [[json-web-token|JWT]] имеет *полезную нагрузку* (пейлоуд, payload) в виде объекта JSON с полями, известными как [[claim|утверждения]] (claims, клеймы)[^2]. И здесь мы добавляем к этому объекту некоторые интересующие нас клеймы.

Чтобы закончить имплементацию интерфейса, добавим метод `IsActiveAsync`:
```csharp
public async Task IsActiveAsync(IsActiveContext context)
{
	var subject = context.Subject.GetSubjectId();
	var user = await usersManager.FindByIdAsync(subject);
	context.IsActive = user != null;
}
```

Теперь зарегистрируем наш класс в [[di-container|контейнере зависимостей]]. Для этого в файле **Startup.cs**[^3] проекта **AuthProvider** в методе `ConfigureServices` добавим
```csharp
services.AddScoped<IProfileService, UserProfileService>();
```
Наконец, убедимся в том, что наше приложение доступно по протоколу https[^4]. Откроем файл **AuthProvider/Properties/launchSettings.json** и убедимся, что `applicationUrl` имеет такой вид (изменим нужную строку, если нужно):
```json
"applicationUrl": "https://localhost::5001;http://localhost:5000"
```
Также нам нужно исправить в файле **AuthProvider/wwwroot/admin/assets/env.js** все URL на `https://localhost::5001`

[^1]: про создание проекта из шаблона также написано [[ch-2-creating-your-first-app|тут]];
[^2]: Про клеймы в ASP.NET Core подробно рассказывается у Эндрю Лока [[ch-14-users-and-claims-in-asp-net-core|здесь]] и [[ch-15-using-policies-for-claims-based-authorization|здесь]];
[^3]: Про класс `Startup` и файл **Startup.cs** рассказывается у Эндрю Лока, например [[ch-2-startup-cs|здесь]];
[^4]: О подробностях работы с протоколом HTTPS в ASP.NET Core Эндрю Лок рассказывает [[ch-18-adding-https-to-app|здесь]]