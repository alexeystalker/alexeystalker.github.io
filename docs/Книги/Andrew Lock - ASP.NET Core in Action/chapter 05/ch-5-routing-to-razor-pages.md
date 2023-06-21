---
share: true
tags:
 - NET/ASPNETCore/routing
 - NET/Razor
---
# Маршрутизация и страницы Razor
Как уже [[ch-5-convention-based-vs-attribute|упоминалось]], страницы Razor используют маршрутизацию на основе атрибутов, создавая [[route-template|шаблоны маршрутов]] на основе соглашений. ASP.NET Core создает шаблон маршрута для каждой страницы Razor в момент вызова метода `MapRazorPages()`:
```csharp
app.UseEndpoints(endpoints =>
{
	endpoints.MapRazorPages();
});
```
Для каждой страницы используется путь к ней относительно папки `Pages/`; расширение .cshtml отбрасывается. То есть, странице `Pages/Products/View.cshtml` соответствует шаблон маршрута `products/view`.
По умолчанию для каждой страницы Razor создается один шаблон маршрута, за исключением страницы Index.cshtml — для них создается 2 шаблона маршрутов, с сегментом `index` и без него. Так, страница `Pages/ToDo/Index.cshtml` получит два маршрута:
- `todo`
- `todo/index`

