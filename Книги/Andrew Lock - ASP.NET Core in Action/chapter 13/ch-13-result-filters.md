---
share: true
tags:
 - NET/ASPNETCore/filters
---
# Фильтры результатов: настройка результатов действий перед их выполнением
Фильтры результатов выполняются непосредственно перед и после выполнения `IActionResult`, возвращаемого [[action-method|методом действия]] или [[ch-13-action-filters|фильтрами действий]]. При этом нужно учитывать, что, если выполнение конвейера было прервано установкой `context.Result` вне фильтра действий или страниц, фильтры результатов не будут выполняться, хотя `IActionResult` выполнится для генерирования ответа.
В ASP.NET Core есть несколько фильтров результатов:
- `ProducesAttribute` — приводит к тому, чторезультат веб-API будет сериализован в определенный выходной формат. Например, `[Produces("application/xml")]` приведет к попытке форматирования в XML, даже если этот формат не указан в заголовке `Accept` запроса;
- `FormatFilterAttribute` — дает указания форматеру искать значение маршрута или параметр строки запроса `format`, и использовать его для определения выходного формата. Например, `/api/recipe/11?format=json` отформатирует результат в JSON, а `/api/recipe/11?format=xml` — в XML.

В примере настроим заголовок `LastModified` при помощи фильтра результатов:
```csharp
public class AddLastModifiedHeaderAttribute : ResultFilterAttribute
{
	public override void OnResultExecuting(ResultExecutingContext context)
	{
		if (context.Result is OkObjectResult result 
		&& result.Value is RecipeDetailViewModel detail)
		{
			var viewModelDate = detail.LastModified;
			context.HttpContext.Response.GetTypedHeaders().LastModified = viewModelDate;
		}
	}
}
```
Также фильтр результатов может реализовать метод `OnResultExecuted()`, который будет выполняться после выполнения `IActionResult`, например, чтобы проверить, не было ли во время выполнения исключений. Как правило, ответ в этом методе изменить уже нельзя.

## Запуск фильтров после прерывания выполнения с помощью интерфейса `IAlwaysRunResultFilter`
Для того, чтобы фильтр результатов выполнялся в любом случае, а не только после успешной работы метода (или фильтров) действия, нужно реализовать интерфейс `IAlwaysRunResultFilter` или `IAsyncAlwaysRunResultFilter`. Такой фильтр будет работать, как и обычный фильтр результатов, при этом срабатывая даже в том случае, когда фильтр авторизации, ресурсов или исключений прервет выполнение конвейера. Подробнее см. в [документации](https://docs.microsoft.com/ru-ru/aspnet/core/mvc/controllers/filters?view=aspnetcore-6.0#ialwaysrunresultfilter-and-iasyncalwaysrunresultfilter)