---
share: true
tags:
 - NET/ASPNETCore/routing
 - NET/MVC
---
# Сочетание атрибутов маршрута, чтобы шаблоны маршрутов следовали принципу DRY
Рассмотрим код с дублирующимися маршрутами.
```csharp
public class CarController
{
	[Route("api/car/start")]
	[Route("api/car/ignition")]
	[Route("/start-car")]
	public IActionResult Start()
	{
		/*реализация*/
	}
	[Route("api/car/speed/{speed}")]
	[Route("/set-speed/{speed}")]
	public IActionResult SetCarSpeed(int speed)
	{
		/*реализация*/
	}
}
```
Здесь `api/car` содержится в большинстве маршрутов. Чтобы исправить ситуацию, можно применить атрибут `[Route]` к классу контроллера:
```csharp
[Route("api/car")]
public class CarController
{
	[Route("start")]
	[Route("ignition")]
	[Route("/start-car")]
	public IActionResult Start()
	{
		/*реализация*/
	}
	[Route("speed/{speed}")]
	[Route("/set-speed/{speed}")]
	public IActionResult SetCarSpeed(int speed)
	{
		/*реализация*/
	}
}
```
В этом случае, если шаблон у действия не начинается с `/`, шаблон действия будет объединением шаблона контроллера и шаблона действия.