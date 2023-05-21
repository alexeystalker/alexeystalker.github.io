---
share: true
tags:
 - NET/Razor/tag-helper
---
# Создание форм с помощью тег-хелперов
Рассмотрим, как использовать тег-хелперы, которые работают с формами.
Предположим, у нас есть модель `UserBindingModel` из [[ch-6-using-data-annotations|главы 6]]:
```csharp
public class UserBindingModel
{
	[Required]
	[StringLength(100)]
	[Display(Name = "Your name")]
	public string FirstName { get; set; }
	
	[Required]
	[StringLength(100)]
	[Display(Name = "Last name")]
	public string LastName { get; set; }
	
	[Required]
	[EmailAddress]
	public string Email { get; set; }
	
	[Phone]
	[Display(Name = "Phone number")]
	public string PhoneNumber { get; set; }
}
```
Атрибуты `DataAnnotations`, использованные для валидации модели, также используются шаблонизатором Razor в качестве метаданных, необходимых для генерации правильного HTML-кода с использованием тег-хелперов.
Используем `UserBindingModel` согласно [[ch-6-organizing-your-binding-models|рекомендациям]]:
```csharp
public class CheckoutModel : PageModel
{
	[BindProperty]
	public UserBindingModel Input { get; set; }
}
```
Используя размеченный класс, тег-хелперы и небольшое количество HTML-кода, получим примерно следующую форму:
![[Pasted image 20220228192941.png]]
Вот какой шаблон для этого нужен:
```razor
@page
@model CheckoutModel
@{
	ViewData["Title"] = "Checkout";
}
<h1>@ViewData["Title"]</h1>
<form asp-page="Checkout">
	<div class="form-group">
		<label asp-for="Input.FirstName"></label>
		<input class="form-control" asp-for="Input.FirstName" />
		<span asp-validation-for="Input.FirstName"></span>
	</div>
	<div class="form-group">
		<label asp-for="Input.LastName"></label>
		<input class="form-control" asp-for="Input.LastName" />
		<span asp-validation-for="Input.LastName"></span>
	</div>
	<div class="form-group">
		<label asp-for="Input.Email"></label>
		<input class="form-control" asp-for="Input.Email" />
		<span asp-validation-for="Input.Email"></span>
	</div>
	<div class="form-group">
		<label asp-for="Input.PhoneNumber"></label>
		<input class="form-control" asp-for="Input.PhoneNumber" />
		<span asp-validation-for="Input.PhoneNumber"></span>
	</div>
	<button type="submit" class="btn btn-primary">Submit</button>
</form>
```
Здесь использованы:
- тег-хелпер Form для элемента `<form>`: он использует маршрутизацию для определения URL-адреса для отправки формы;
- тег-хелпер Label для элементов `<label>`: он использует DataAnnotations для определения заголовка;
- тег-хелпер Input для элементов `<input>`: он использует DataAnntoations для определения типа ввода;
- тег-хелпер Validation для элементов валидации `<span>`: использует DataAnnotations для определения текста сообщения об ошибке валидации.

*Дальше в книге приведен HTML-код, в который разворачиваются тег-хелперы, но я не буду его приводить, он очень большой.*

---
Рассмотрим эти и другие тег-хелперы отдельно.
#### [[ch-8-form-tag-helper|Тег-хелпер формы]]
#### [[ch-8-label-tag-helper|Тег-хелпер метки (label)]]
#### [[ch-8-input-and-textarea-tag-helpers|Тег-хелперы input и textarea]]
#### [[ch-8-select-tag-helper|Тег-хелпер раскрывающегося списка]]
#### [[ch-8-validation-message-and-validation-summary-tag-helpers|Тег-хелперы сообщений валидации и сводки сообщений]]