---
share: true
tags:
 - NET/Razor
 - NET/Blazor
---
# Компоненты представления: добавление логики в частичные представления
*Компоненты представления (view components)* — отдельные сущности, позволяющие инкапсулировать как логику отрисовки, так и бизнес-логику для отображения небольшого раздела страницы. Они вызываются непосредственно из представления Razor, а не в ответ на HTTP-запрос.
> [!Note]- Компоненты представления в сравнении с Razor Components и Blazor
> В Blazor есть две модели программирования — на стороне клиента и стороне сервера — и в обоих используются *компоненты* Blazor (официально названные *компонентами Razor*). Их не следует путать с компонентами представления — они не взаимодействуют с тег-хелперами или компонентами представления, и их сложно комбинировать с формами Razor Page.
> Однако при необходимости можно встроить компонент Blazor внутрь страницы Razor, как показано [тут](https://learn.microsoft.com/en-us/aspnet/core/blazor/components/prerendering-and-integration?view=aspnetcore-6.0&pivots=webassembly#render-components-in-a-page-or-view-with-the-component-tag-helper). Также можно использовать компоненты Blazor как способ заменить вызовы AJAX, как показано [тут, в блоге автора](https://andrewlock.net/replacing-ajax-calls-in-razor-pages-using-razor-components-and-blazor/).
> Однако, если интерактивность на стороне клиента не нужна, лучше использовать компоненты представления. Дополнительная информация — [в ещё одной записи из блога автора](https://andrewlock.net/dont-replace-your-view-components-with-razor-components/)

Создадим специальный компонент представления для нашего [[ch-12-sample-application|приложения с рецептами]], который будет отображать список ссылок на недавно созданные рецепты в случае, когда пользователь выполнил вход, и форму для входа в противном случае.
> [!tip] Совет
> Используйте [[ch-7-using-partial-views-to-encapsulate-markups|частичные представления]] для инкапсуляции отрисовки [[view-model|модели представления]] или её части; если же логика отрисовки требует доступ к БД или бизнес-логике, или если раздел логически отличается от содержимого главной страницы, желательно использовать компонент представления.

Компоненты представления вызываются непосредственно из представлений и макетов Razor с использованием синтаксиса в стиле тег-хелпера с префиксом `vc`:
```html
<vc:my-recipes number-of-recipes="3">
</vc:my-recipes>
```
Специальные компоненты обычно реализуют, унаследовавшись от базового класса `ViewComponent` и реализуют метод `InvokeAsync()`. Параметры, передаваемые в `InvokeAsync()`, соответствуют свойствам элемента тег-хелпера (с заменой camelCase на kebabCase):
```csharp
public class MyRecipesViewComponent: ViewComponent
{
	private readonly RecipeService _recipeService;
	private readonly UserManager<ApplicationUser> _userManager;
	
	public MyRecipesViewComponent(
		RecipeService recipeService, UserManager<ApplicationUser> userManager)
	{
		_recipeService = recipeService;
		_userManager = userManager;
	}
	
	public async Task<IViewComponentResult> InvokeAsync(int numberOfRecipes)
	{
		if(!User.Identity.IsAuthenticated)
		{
			return View("Unauthenticated");
		}
		var userId = _userManager.GetUserId(HttpContext.User);
		var recipes = await _recipeService.GetRecipesForUser(
			userId, numberOfRecipes);
			
		return View(recipes);
	}
}
```
Для задания имени компонента представления можно использовать атрибут `[ViewComponent]`.
Метод `InvokeAsync()` должен вернуть `Task<IViewComponentResult>`; компоненты представления не могут возвращать коды состояния или перенаправления, они *должны* как-то визуализировать содержимое. Поэтому необходимо использовать либо метод `View()`, либо `Content()`, который кодирует содержимое в HTML и отображает его напрямую.
Компоненты представления имеют доступ к `HttpContext` и текущему запросу.
Частичные представления для компонентов работают [[ch-7-using-partial-views-to-encapsulate-markups|аналогично частичным представлениям Razor]], но хранятся отдельно, в одном из следующих мест:
- **Views/Shared/Components/ComponentName/TemplateName**;
- **Pages/Shared/Components/ComponentName/TemplateName**.

Для нашего компонента необходимо два представления (используем папку Pages):
- **Pages/Shared/Components/MyRecipes/Default.cshtml**;
- **Pages/Shared/Components/MyRecipes/Unauthenticated.cshtml**.

Также при использовании компонентов представления необходимо помнить о следующем:
- классы компонентов представления должны быть открытыми, невложенными и неабстрактными;
- с ними нельзя использовать фильтры;
- можно использовать макеты, а также `@sections`, как показано [[ch-7-overriding-parent-layouts-using-sections|здесь]], но данные секции не зависят от основного макета Razor;
- компоненты представления изолированы от страницы Razor, на которой они отображаются;
- при использовании тег-хелпера `<vc:my-recipes>` необходимо [[ch-20-printing-environment-info-with-custom-tag-helper#Регистрация тег-хелпера|импортировать]] его в качестве специального тег-хелпера;
- вместо использования синтаксиса тег-хелпера можно вызвать компонент представления напрямую из представления:
	```razor
	@await Component.InvokeAsync("MyRecipes", new {numberOfRecipes = 3})
	```
