---
share: true
tags:
 - NET/Razor/tag-helper
---
# Редакторы кода и тег-хелперы
Одной из главных мотиваций для создания тег-хелперов была необходимость поддержки сторонних редакторов кода, прежде всего — HTML кода. Поэтому тег-хелперы легко интегрируются в обычный синтаксис HTML, добавляя элементы, которые выглядят как атрибуты (обычно с префиксом `asp-*`)
Рассмотрим пример шаблона формы с использованием тег-хелперов.
```razor
@page
@model ConvertModel
<form method="post">
	<div class="form-group">
		<label asp-for="CurrencyFrom"></label>
		<input class="form-control" asp-for="CurrencyFrom" />
		<span asp-validation-for="CurrencyFrom"></span>
	</div>
	<div class="form-group">
		<label asp-for="Quantity"></label>
		<input class="form-control" asp-for="Quantity" />
		<span asp-validation-for="Quantity"></span>
	</div>
	<div class="form-group">
		<label asp-for="CurrencyTo"></label>
		<input class="form-control" asp-for="CurrencyTo" />
		<span asp-validation-for="CurrencyTo"></span>
	</div>
	<button type="submit" class="btn btn-primary">Submit</button>
</form>
```
Вот какие здесь тег-хелперы:
- `asp-for` с тегами `<label>` генерирует заголовок для ярлыков на основе [[view-model|модели представления]], а именно атрибута `[Display]`;
- `asp-for` с тегами `<input>` генерирует правильный тип, значение, имя и атрибуты валидации на основе DataAnnotations и типа указанного свойства;
- `asp-validation-for` записывает сообщения о валидации, взятые из `ModelState`, в теги `<span>`.