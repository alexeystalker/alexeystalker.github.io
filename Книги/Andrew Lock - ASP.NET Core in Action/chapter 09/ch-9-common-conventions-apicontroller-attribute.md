---
share: true
title: Использование общепринятых соглашений с атрибутом [ApiController]
tags:
 - NET/MVC
 - API
---
# Использование общепринятых соглашений с атрибутом \[ApiController\]
Атрибут `[ApiController]` упрощает процесс создания контроллеров веб-API.
Рассмотрим код контроллера без атрибута:
```csharp
public class FruitController : ControllerBase
{
	List<string> _fruit = new List<string>
	{
		"Pear", "Lemon", "Peach"
	};
	[HttpPost("fruit")]
	public IActionResult Update([FromBody] UpdateModel model)
	{
		if(!ModelState.IsValid)
		{
			return BadRequest(new ValidationProblemDetails(ModelState));
		}
		if(model.Id < 0 || model.Id > _fruit.Count)
		{
			return NotFound(new ProblemDetails()
			{
				Status = 404,
				Title = "Not Found",
				Type = "https://tools.ietf.org/html/rfc7231#section-6.5.4"
			});
		}
		_fruit[model.Id] = model.Name;
		return Ok();
	}
	
	public class UpdateModel
	{
		public int Id { get; set; }
		[Required]
		public string Name { get; set; }
	}
}
```
Здесь:
- атрибут `[FromBody]` используется для того, чтобы тело запроса читалось как JSON, а не значения формы
- Если данные не проходят валидацию, возвращается BadRequest с деталями в виде объекта `ValidationProblemDetails`
- При возврате `404 Not Found` возращается объект `ProblemDetails`

Теперь рассмотрим тот же код, но с атрибутом `[ApiController]`:
```csharp
[ApiController]
public class FruitController : ControllerBase
{
	List<string> _fruit = new List<string>
	{
		"Pear", "Lemon", "Peach"
	};
	[HttpPost("fruit")]
	public IActionResult Update(UpdateModel model)
	{
		if(model.Id < 0 || model.Id > _fruit.Count)
		{
			return NotFound();
		}
		_fruit[model.Id] = model.Name;
		return Ok();
	}
	
	public class UpdateModel
	{
		public int Id { get; set; }
		[Required]
		public string Name { get; set; }
	}
}
```
Атрибут `[ApiController]` автоматически применяет к контроллеру несколько соглашений:
- *маршрутизация на основе атрибутов*;
- *автоматические ответы с кодом 400* — `ModelState.IsValid` проверяется за нас, при помощи [[ch-13-action-filters|фильтра]]; 
- *вывод источника привязки модели* — при наличии атрибута `[ApiController]` атрибут `[FromBody]` предполагается по умолчанию для сложных параметров;
- *`ProblemDetails` для кодов ошибок*. Атрибут `[ApiController]` перехватывает коды ошибок и оборачивает их в `ProblemDetails`

Использование `ProblemDetails`[^1] для возврата ошибок — ключевая особенность атрибута `[ApiController]`.
Для того, чтобы оборачивать в `ProblemDetails` ошибки, случающиеся в процессе работы конвейера промежуточного ПО, нужно воспользоваться одним из сторонних пакетов. Подробнее вопрос разбирается [в блоге автора](https://andrewlock.net/handling-web-api-exceptions-with-problemdetails-middleware/)

Если мы хотим отключить автоматические ответы с кодом 400 на ошибки валидации - это можно сделать следующим образом в файле Startup.cs:
```csharp
public class Startup
{
	public void ConfigureServices(IServiceCollection services)
	{
		services.AddControllers()
			.ConfigureApiBehaviorOptions(options =>
			{
				options.SuppressModelStateInvalidFilter = true;
			});
	}
}
```

[^1]:Спецификация [здесь](https://datatracker.ietf.org/doc/html/rfc7807)