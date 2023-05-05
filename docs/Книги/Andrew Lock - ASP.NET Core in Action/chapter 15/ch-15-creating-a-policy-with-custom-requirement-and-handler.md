---
share: true
tags:
 - NET/ASPNETCore
 - NET/authorization
 - security
---
# Создание политики со специальным требованием и обработчиком
Рассмотрим реализацию [[ch-15-creating-custom-policies|политики]] `"CanAccessLounge"`. (Составляющие политики - на рисунке)
![[Pasted image 20220621191024.png]]
## Создание IAuthorizationRequirement для представления требования
В терминах ASP.NET Core требование — это любой класс, реализующий интерфейс `IAuthorizationRequirement`. Это интерфейс-маркер, не имеющий методов.
Обычно, требования — это просто POCO-классы. Например, `AllowedInLoungeRequirement` может вообще не иметь методов или свойств:
```csharp
public class AllowedInLoungeRequirement : IAuthorizationRequirement { }
```
Часто к требованию добавляют одно или несколько свойств, чтобы сделать его более универсальным. Например:
```csharp
public class MinimumAgeRequirement : IAuthorizationRequirement
{
	public MinimumAgeRequirement(int minimumAge)
	{
		MinimumAge = minimumAge;
	}
	public int MinimumAge { get; }
}
```
Обработчики могут использовать свойства требования, чтобы определить, выполнено ли оно.
## Создание политики с несколькими требованиями
Теперь настроим политику с этими требованиями:
```csharp
public void ConfigureServices(IServiceCollection services)
{
	...
	services.AddAuthorization(options =>
	{
		options.AddPolicy( //Добавим простую политику
			"CanEnterSecurity",
			policyBuilder => policyBuilder
				.RequireClaim(Claims.BoardingPassNumber));
		
		options.AddPolicy(
			"CanAccessLounge",
			policyBuilder => policyBuilder.AddRequirements(
				new MinimumAgeRequirement(18),
				new AllowedInLoungeRequirement()
			));
	});
	...
}
```
Теперь, когда мы добавили политики, мы можем использовать их  названия в атрибуте `[Authorize]`:
```csharp
[Authorize("CanAccessLounge")]
public class AirportLoungeModel : PageModel
{
	public void OnGet() { }
}
```
## Создаём обрабочики, чтобы отвечать требованиям
Обработчики авторизации содержат логику того, как соответствовать конкретной реализации `IAuthorizationRequirement`. При выполнении обработчик может делать одно из трёх:
- отметить обработку требований как успешную;
- ничего не делать;
- явно не отвечать требованию.

Обработчики должны быть унаследованы от `AuthorizaionHandler<T>`, где `T` - тип требования, которое они обладают:
```csharp
public class FrequentFlyerHandler : AuthorizationHandler<AllowedInLoungeRequirement>
{
	protected override Task HandleRequirementAsync(
		AuthorizationHandlerContext context,
		AllowedInLoungeRequirement requirement)
	{
		if (context.User.HasClaim("FrequentFlyerClass", "Gold"))
		{
			context.Succeed(requirement);
		}
		return Task.CompletedTask;
	}
}
```
В случае успеха обработчик помечает требование как успешно выполненное, вызывая метод `context.Succeed()` и передавая требование в качестве аргумента. Если у пользователя *нет* утверждения — обработчик ничего не делает.
Такое поведение необходимо для реализации [[ch-15-the-building-blocks-of-policy|OR логики]] при композиции обработчиков.
Вот еще один обработчик для `AllowedInLoungeRequirement`:
```csharp
public class IsAirlineEmployeeHandler : AuthorizationHandler<AllowedInLoungeRequirement>
{
	protected override Task HandleRequirementAsync(
		AuthorizationHandlerContext context,
		AllowedInLoungeRequirement requirement)
	{
		if (context.User.HasClaim(c => c.Type == "EmployeeNumber"))
		{
			context.Succeed(requirement);
		}
		return Task.CompletedTask;
	}
}
```
Для `MinimumAgeRequirement` напишем обработчик, проверяющий, сколько лет прошло с момента рождения пользователя, и соответствует ли он требованию:
```csharp
public class MinimumAgeHandler : AuthorizationHandler<MinimumAgeRequirement>
{
	protected override Task HandleRequirementAsync(
		AuthorizationHandlerContext context,
		MinimumAgeRequirement requirement)
	{
		var dateOfBirthStr = context.User.FindFirstValue(ClaimTypes.DateOfBirth);
		
		if (dateOfBirthStr == null)
		{
			return Task.CompletedTask;
		}
		
		var dateOfBirth = Convert.ToDateTime(dateOfBirthStr);
		var cutoff = dateOfBirth.AddYears(requirement.MinimumAge);
		
		if (cutoff < DateTime.Today)
		{
			context.Succeed(requirement);
		}
		
		return Task.CompletedTask;
	}
}
```

Иногда возможны ситуации, в которых мы хотим *гарантировать* неудачу проверки всей политики, несмотря на то, насколько были успешны другие обработчики (и требования). Для этого нужно вызвать метод `context.Fail()`, чтобы явно сделать авторизацию неудачной:
```csharp
public class BannedFromLoungeHandler : AuthorizationHandler<AllowedInLoungeRequirement>
{
	protected override Task HandleRequirementAsync(
		AuthorizationContext context,
		AllowedInLoungeRequirement requirement)
	{
		if (context.User.HasClaim(c => c.Type == "IsBanned"))
		{
			context.Fail();
		}
		return Task.CompletedTask;
	}
}
```
> [!Note] Примечание
> Система авторизации всегда будет выполнять все обработчики для требования, и все требования к политике, независимо, вызывает ли какой-либо обработчик метод `Succeed()` или `Fail()`.

Теперь надо зарегистрировать наши обработчики в контейнере внедрения зависимостей:
```csharp
public void ConfigureServices(IServiceCollection services)
{
	...
	services.AddAuthorization(options =>
	{
		options.AddPolicy( //Добавим простую политику
			"CanEnterSecurity",
			policyBuilder => policyBuilder
				.RequireClaim(Claims.BoardingPassNumber));
		
		options.AddPolicy(
			"CanAccessLounge",
			policyBuilder => policyBuilder.AddRequirements(
				new MinimumAgeRequirement(18),
				new AllowedInLoungeRequirement()
			));
	});
	services.AddSingleton<IAuthorizationHandler, MinimumAgeHandler>();
	services.AddSingleton<IAuthorizationHandler, FrequentFlyerHandler>();
	services.AddSingleton<IAuthorizationHandler, BannedFromLoungeHandler>();
	services.AddSingleton<IAuthorizationHandler, IsAirlineEmployeeHandler>();
	...
}
```