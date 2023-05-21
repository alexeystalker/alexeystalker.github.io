---
share: true
tags:
 - NET/Razor
---
# Представления: визуализация пользовательского интерфейса
![[Pasted image 20220210202045.png]]
Типичный запрос следует этапам, показанным на рисунке:
1. Конвейер промежуточного ПО получает запрос, а компонент маршрутизации определяет конечную точку для вызова;
2. Связыватель модели использует запрос для сборки [[binding-model|моделей привязки]] страницы;
3. Предполагая, что всё в порядке, [[page-handler|обработчик страницы]] обращается к сервисам модели приложения, в процессе генерируются значения для передачи в представление путём передачи в свойства `PageModel`;
4. Шаблон представления Razor использует `PageModel` для генерации окончательного ответа, возвращаемого через конвейер промежуточного ПО.

Представления не должны вызывать методы `PageModel` — как правило, представление должно получать доступ только к данным через свойства.

Обработчики страниц Razor указывают на то, что представление Razor следует визуализировать, [[ch-4-actionresults-responses|возвращая объект]] `Pageresult` (или `void`). Инфраструктура Razor Pages выполняет представление Razor, связанное с данной страницей Razor, для генерации окончательного ответа. Для управления динамической генерацией используется код C\#.
Например, пусть мы хотим добавить страницу со списком пользователей приложения, а также просматривать детали пользователя или создавать нового.
![[Pasted image 20220210203638.png]]
Вот пример шаблона для генерации интерфейса с картинки. Он сочетает HTML-код с C\# и тег-хелперами.
```razor
@page
@model IndexViewModel
<div class="row">
	<div class="col-md-6">
		<form method="post">
			<div class="form-group">
				<label asp-for="NewUser"></label>
				<input class="form-control" asp-for="NewUser" />
				<span asp-validation-for="NewUser"></span>
			</div>
			<div class="form-group">
				<button type="submit" class="btn btn-success">Add</button>
			</div>
		</form>
	</div>
</div>

<h4>Number of users: @Model.ExistingUsers.Count</h4>
<div class=row>
	<div class="col-md-6">
		<ul class="list-group">
			@foreach (var user in Model.ExistingUsers)
			{
			<li class="list-group-item d-flex justify-content-between">
				<span>@user</span>
				<a class="btn btn-info" 
				   asp-page="ViewUser" 
				   asp-route-username="@user">View</a>
			</li>				
			}
		</ul>
	</div>
</div>
```
В этом примере демонстрируются различные функции Razor. Здесь есть и HTML-код, и C\#-инструкции, используемые для динамической генерации, а также тег-хелперы, выглядящие как обычные атрибуты, начинающиеся с `asp-`, и являющиеся частью языка Razor.
Страницы Razor компилируются при сборке приложения, однако также можно активировать компиляцию страниц во время выполнения.[^1]

[^1]:Подробные сведения см. в [документации](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/view-compilation?view=aspnetcore-5.0&tabs=visual-studio)