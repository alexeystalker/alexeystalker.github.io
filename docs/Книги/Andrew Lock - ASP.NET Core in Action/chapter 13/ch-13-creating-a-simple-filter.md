---
share: true
tags:
 - NET/ASPNETCore/filters
---
# Создание простого фильтра
Создадим самые примитивные фильтры — делающие некий вывод в консоль.

Фильтр определенного типа реализуется с помощью одного из пары — синхронного и асинхронного — интерфейсов:
- *фильтры авторизации* — `IAuthorizationFilter` и `IAsyncAuthorizationFilter`;
- *фильтры ресурсов* — `IResourceFilter` и `IAsyncResourceFilter`;
- *фильтры действий* — `IActionFilter` и `IAsyncActionFilter`;
- *фильтры страниц* — `IPageFilter` и `IAsyncPageFilter`;
- *фильтры исключений* — `IExceptionFilter` и `IAsyncExceptionFilter`;
- *фильтры результатов* — `IResultFilter` и `IAsyncResultFilter`.

Обычно фильтры реализуются как атрибуты C#, что позволяет использовать их для декорирования контроллеров, действий и страниц Razor Pages.
Следует использовать один интерфейс из пары; если будут реализованы оба интерфейса, использоваться будет асинхронный.

Рассмотрим пример реализации `IResourceFilter`.
```csharp
public class LogResourceFilter : Attribute, IResourceFilter
{
	public void OnResourceExecuting(ResourceExecutingContext context)
	{
		Console.WriteLine("Executing!");
	}
	public void OnResourceExecuted(ResourceExecutedContext context)
	{
		Console.WriteLine("Executed!");
	}
}
```
Метод `OnResourceExecuting` вызывается, когда запрос сначала достигает этапа обработки, на котором вызывается фильтр ресурсов. Метод `OnResourceExecuted` вызывается после выполнения остальной части конвейера[^1]. Такие же методы есть и в интерфейсах других типов фильтров.

Информация, доступная на момент выполнения метода, содержится в аргументе: так, `ResourceExecutingContext` содержит объект `HttpContext`, сведения о маршруте, который выбрал это действие, детали действия и т.д. Для более поздних фильтров контексты будут содержать дополнительные подробности, такие как аргументы метода действия и `ModelState`. Для методов `*Executed` контекст также содержит информацию о том, как исполнялась остальная часть конвейера.

Асинхронная версия фильтра ресурсов  требует реализации одного метода.
```csharp
public class LogAsyncResourceFilter : Attribute, IAsyncResourceFilter
{
	public async Task OnResourceExecutionAsync(
		ResourceExecutingContext context,
		ResourceExecutionDelegate next)
	{
		Console.WriteLine("Executing async!");
		ResourceExecutedContext executedContext = await next();
		Console.WriteLine("Executed async!");
	}
}
```
Здесь вторым аргументом передается делегат, который нужно вызвать (асинхронно), чтобы выполнить оставшуюся часть конвейера.

[^1]:См. картинки [[ch-13-mvc-filter-pipeline|здесь]] и [[ch-13-razor-pages-filter-pipeline|здесь]]