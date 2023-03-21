---
share: true
tags:
 - NET/SignalR
 - NET/authorization
---
# Применяем авторизацию в SignalR
Аутентификация сама по себе - это хорошо, но далеко не всегда эффективно. Необходимо быть уверенным не только в том, что доступ к системе имеют известные пользователи, но и в том, что пользователям доступны только те ресурсы, к которым им разрешено иметь доступ. Именно за эту часть отвечает авторизация.
В ASP.NET Core cуществует[^1] несколько различных типов авторизации, и все они применимы к SignalR. Необходимо только сконфигурировать авторизационные обработчики. Существует множество путей сделать это, и мы рассмотрим некоторые из них.
## Создание особого требования
Одним из способов является реализация[^2] особого класса, унаследованного от `AuthorizationHandler`. Создадим в проекте SignalRServer класс `RoleRequirement.cs` со следующим содержимым:
```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.SignalR;
using System.Security.Claims;

namespace SignalRServer
{
	public class RoleRequirement: AuthorizationHandler<RoleRequirement, HubInvocationContext>,
		IAuthorizationRequirement
	{
		private readonly string requiredRole;
		public RoleRequirement(string requiredRole)
		{
			this.requiredRole = requiredRole;
		}
		
		protected override Task HandleRequirementAsync(
			AuthorizationHandlerContext context,
			RoleRequirement requirement,
			HubInvocationContext resource)
		{
			var roles = ((ClaimsIdentity)context.User.Identity).Claims
				.Where(c => c.Type == ClaimTypes.Role)
				.Select(c => c.Value);

			if(roles.Contains(requiredRole))
				context.Succeed(requirement);

			return Task.CompletedTask;
		}
	}
}
```
Когда этот класс будет зарегистрирован в промежуточном ПО авторизации, метод `HandleRequirementAsync` будет вызван для запроса, из которого уже был выделен аутентификационный токен. В данном случае мы проверяем, что требуемая роль присутствует среди клеймов этого токена.
Видно, что наш класс использует `HubInvocationContext` в качестве одного из параметров типа. В нашем коде он не используется, но показывает, что это требование будет применено в контексте запроса к хабу SignalR. При этом это не просто маркер - можно реализовать некоторую логику, использующую параметр `resource` этого типа.
## Конфигурируем промежуточное ПО авторизации
Добавляем авторизацию к сервисам:
```csharp
builder.Services.AddAuthorization(options =>
{
	options.AddPolicy("BasicAuth", policy => policy.RequireAuthenticatedUser());
	options.AddPolicy("AdminClaim", policy => policy.RequireClaim("admin"));
	options.AddPolicy("AdminOnly", policy => policy.Requirements.Add(new RoleRequirement("admin")));
})
```
В этом коде показаны различные способы добавления [[authorization-policy|политик авторизации]][^3].
Первая политика, `"BasicAuth"`, требует, чтобы пользователи были аутентифицированы. Практически то же самое, что и использование атрибута `[Authorize]` без указания политики.
Вторая политика, `"AdminClaim"`, требует, чтобы у пользователя был клейм `admin`, при этом неважно, какое значение будет у этого клейма. Также у метода `RequireClaim` есть перегрузка с двумя параметрами, второй параметр представляет собой массив допустимых значений клейма.
Наконец, третья политика, `"AdminOnly"` - это политика, где мы применяем наше особое требование. В данном случае, требование выполнится, если у пользователя есть клейм `role` и его значение равно `admin`.
## Применение правил авторизации к конкретным конечным точкам
Начнем с того, что изменим атрибут, который мы применили к классу `LearningHub`.
```csharp
[Authorize(AuthenticationSchemes = JwtBearerDefaults.AuthenticationScheme + "," + CookieAuthenticationDefaults.AuthenticationScheme, Policy = "BasicAuth")]
```
Таким образом, политику можно применить к любой конечной точке в ASP.NET Core. Что касается поведения, то оно не изменилось по сравнению с ситуацией, когда мы не задавали политику явно.
Далее, добавим следующий атрибут к методу хаба `SendToGroup`:
```csharp
[Authorize(Roles = "user")]
```
Здесь мы не применили именованных политик. Мы указали, что метод можно выполнить только пользователю с ролью `user`, то есть, другими словами, роль `user` есть среди ролей, указанных в клейме `role` пользователя.
Наконец, добавим к методу `AddUserToGroup` такой атрибут:
```csharp
[Authorize("AdminOnly")]
```
Здесь мы передали заданное имя политики.



[^1]: См главу об авторизации у [[ch-15-authorization|Эндрю Лока]];
[^2]: [[ch-15-creating-a-policy-with-custom-requirement-and-handler|Подробнее]];
[^3]: Подробнее о политиках авторизации [[ch-15-using-policies-for-claims-based-authorization|здесь]] и далее по ссылкам