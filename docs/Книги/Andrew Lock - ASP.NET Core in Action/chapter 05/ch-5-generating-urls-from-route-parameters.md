---
share: true
tags:
 - NET/ASPNETCore/routing
 - URL
 - NET/Razor
 - NET/MVC
---
# Генерация URL-адресов из параметров маршрута
Инфраструктура маршрутизации может использоваться и для динамической генерации URL-адресов во время выполнения. По сути, это процесс, обратный сопоставлению адреса запроса.
## Создание URL-адресов для страницы Razor
Создадим ссылку на страницу `Pages/Currency/View.cshtml`:
```csharp
public class IndexModel : PageModel
{
	public void OnGet()
	{
		var url = Url.Page("Currency/View", new { code = "USD" });
	}
}
```
Здесь `Url` — свойство в классе `PageModel`, экземпляр `IUrlHelper`. Можно указать *относительный* путь к файлу, или *абсолютный* (относительно папки Pages), начав путь с символа `/` (`"/Currency/View"`). Если в [[route-template|шаблоне маршрута]] страницы явно указан параметр (`"Currency/View/{code}"`), то значение будет включено в [[url-path|путь]] (`"/currency/view/usd"`), иначе значение будет добавлено в [[url-query-string|строку запроса]] (`"/currency/view?code=USD"`).
## Создание URL-адресов для контроллера MVC
Создадим ссылку на [[action-method|метод действия]] `View` в контроллере `Currency`:
```csharp
public class CurrencyController : Controller
{
	[HttpGet("currency/index")]
	public IActionResult Index()
	{
		var url = Url.Action("View", "Currency", new { code = "USD" });
		return Content($"The URL is {url}");
	}
	
	[HttpGet("currency/view/{code}")]
	public IActionResult View(string code)
	{
		/*реализация метода*/
	}
}
```
Здесь `Url` — свойство в классе Controller. В метод `Action` передается имя метода действия и имя контроллера, а также значение маршрута. Вместо использования строки для имени метода действия целесообразно использовать `nameof`.
Если маршрутизация выполняется к методу того же контроллера, то можно использовать перегруженный метод `Action` без указания имени контроллера. IUrlHelper при построении адреса использует *внешние значения* (значения маршрута для текущего запроса), заменяя их предоставляемыми значениями.
## Создание URL-адресов с помощью ActionResults
Часто возникает ситуация, когда URL-адрес не нужно отображать, а следует лишь перенаправить пользователя на новую страницу. Для этого можно использовать `ActionResult` для генерации URL-адреса.
```csharp
public class CurrencyModel : PageModel
{
	public IActionResult OnGetRedirectToPage()
	{
		return RedirectToPage("Currency/View", new { id = 5 });
	}
}
```
Здесь сгенерированный адрес передается в качестве цели для редиректа.
## Создание URL-адресов из других частей вашего приложения
Если требуется сгенерировать URL-адрес извне MVC-контроллера или обработчика страницы Razor, необходимо воспользоваться классом `LinkGenerator`[^1]

[^1]:Документация на LinkGenerator [здесь](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.routing.linkgenerator?view=aspnetcore-5.0)

```csharp
public class SomeClass
{
	private readonly LinkGenerator _link;
	public SomeClass(LinkGenerator linkGenerator)
	{
		_link = linkGenerator;
	}
	
	public void SomeMethod(HttpContext httpContext)
	{
		var url1 = _link.GetPathByPage(httpContext, "/Currency/View", values: new { id = 5 });
		var url2 = _link.GetPathByPage("/Currency/View", values: new { id = 5 });
		var absoluteUrl = _link.GetUriByPage(
			page: "/Currency/View",
			handler: null,
			values: new { id = 5 },
			scheme: "https",
			host: new HostString("example.com");
		)
	}
}
```
Важно! Если в процессе генерации что-то пойдёт не так, LinkGenerator вернет `null`, поэтому нужно проверять результат перед использованием.