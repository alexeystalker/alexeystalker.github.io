---
share: true
tags:
 - NET/ASPNETCore/routing
 - NET/MVC
---
# Маршрутизация на основе атрибутов: Связывание методов действий с URL-адресами
В контроллерах API шаблоны маршрутизации указываются в атрибутах маршрутизации. Каждый [[action-method|метод действия]] декорируется атрибутом, в котором указывается ассоциированный шаблон маршрута:
```csharp
public class HomeController : Controller
{
	[Route("")] //Действие будет выполнено при запросе к адресу /
	public IActionResult Index()
	{
		/*реализация*/
	}
	[Route("contact")] //Действие будет выполнено призапросе к адресу /contact
	public IActionResult Contact()
	{
		/*реализация*/
	}
}
```
Каждый атрибут `[Route]` определяет шаблон маршрута, который должен быть ассоциирован с методом действия. Атрибутов `RouteAttribute` может быть более одного; в этом случае один метод будет соответствовать нескольким адресам:
```csharp
public class CarController
{
	[Route("car/start")]
	[Route("car/ignition")]
	[Route("start-car")]
	public IActionResult Start()
	{
		/*реализация*/
	}
	[Route("car/speed/{speed}")]
	[Route("set-speed/{speed}")]
	public IActionResult SetCarSpeed(int speed)
	{
		/*реализация*/
	}
}
```
Синтаксис шаблонов маршрута аналогичен рассмотренному [[ch-5-route-template-syntax|здесь]].
#### [[ch-9-combining-route-attributes|Сочетание атрибутов маршрута]]
#### [[ch-9-token-replacement|Использование замены маркера]]
#### [[ch-9-handling-http-verbs|Обработка HTTP-методов]]