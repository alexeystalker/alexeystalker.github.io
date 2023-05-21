---
share: true
tags:
 - NET/ASPNETCore/testing
 - testing
---
# Модульное тестирование API-контроллеров
Модульные тесты изолируют поведение; нам нужно протестировать логику, содержащуюся в самом компоненте, отдельно от поведения зависимостей.
Контроллеры WebApi или `PageModels` страниц Razor обычно несут ответственность за следующее:
- в случае недействительного запроса (например, не прошедшего валидацию), возвращают соответствующий `ActionResult` или повторно отображают форму;
- в случа действительного запроса вызывают необходимые сервисы бизнес-логики и возвращают соответствующий `ActionResult` или показывают либо перенаправляют на страницу об успешном результате;
- при необходимости применяют авторизацию.

В качестве примера рассмотрим простой контроллер API:
```csharp
[Route("api/[controller]")]
public class CurrencyController : ControllerBase
{
	private readonly CurrencyConverter _converter = new CurrencyConverter();
	
	[HttpPost]
	public ActionResult<decimal> Convert(InputModel model)
	{
		if(!ModelState.IsValid)
		{
			return BadRequest(ModelState);
		}
		decimal result = _converter.ConvertToGbp(model);
		return result;
	}
}
```
Для начала рассмотрим успешный путь, когда контроллер получил валидный запрос.
```csharp
public class CurrencyControllerTest
{
	[Fact]
	public void Convert_ReturnsValue()
	{
		var controller = new CurrencyController();
		var model = new ConvertInputModel
		{
			Value = 1,
			ExchangeRate = 3,
			DecimalPlaces = 2
		};
		
		ActionResult<decimal> result = controller.Convert(model);
		Assert.NotNull(result);
	}
}
```
Тут важно отметить, что мы тестируем не ответ, который отправляется пользователю, а *только лишь* возвращаемое значение действия, так как процесс сериализации обрабатывается не контроллером, а [[ch-9-generating-response-from-a-model|инфраструктурой форматтера MVC]].
Теперь рассмотрим случай, когда модель не валидна. При обычной работе приложения свойство `ModelState` устанавливается в процессе привязки модели. Так как в модульном тесте процесса привязки нет, это свойство нужно задать вручную.
Вот как можно протестировать путь ошибки валидации:
```csharp
[Fact]
public void Convert_ReturnsBadRequestWhenInvalid()
{
	var controller = new CurrencyController();
	var model = new ConvertInputModel
	{
		Value = 1,
		ExchangeRate = -2,
		DecimalPlaces = 2
	};
	controller.ModelState.AddModelError(
		nameof(model.ExchangeRate),
		"Exchange rate must be greater than zero");
	
	ActionResult<decimal> result = controller.Convert(model);

	Assert.IsType<BadRequestObjectResult>(result.Result);
}
```
> [!note] Примечание
> В предыдущем примере мы действительно передали *невалидную* модель; однако можно было передать и *валидную*, и вообще `null`, так как код контроллера не использует модель в случае, когда `ModelState` содержит ошибку валидации.

В этом разделе мы обсудили, в основном, контроллеры MVC; о модульном тестировании страниц Razor можно прочесть [отдельную статью в документации](https://learn.microsoft.com/ru-ru/aspnet/core/test/razor-pages-tests?view=aspnetcore-6.0).
