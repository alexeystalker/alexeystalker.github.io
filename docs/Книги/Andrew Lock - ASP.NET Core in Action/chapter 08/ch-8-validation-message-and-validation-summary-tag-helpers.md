---
share: true
tags:
 - NET/Razor/tag-helper
 - NET/ASPNETCore/validation
---
# Тег-хелперы сообщений валидации и сводки сообщений
Мы уже [[ch-8-input-and-textarea-tag-helpers|видели]], как тег-хелпер `<input>` генерирует атрибуты валидации `data-val-*`. Однако для того, чтобы отображать сообщения проверки, нужно воспользоваться тег-хелпером для тега `<span>`, используя для этого атрибут `asp-validation-for`:
```html
<span asp-validation-for="Email"></span>
```
когда ошибка возникает на стороне клиента, сообщение будет отображаться в соответствующем теге `<span>`. Если валидация на стороне сервера[^1] завершиться неудачей, этот же элемент будет использоваться для отображения сообщений валидации.

[^1]: О валидации на стороне сервера см. [[ch-6-validating-on-server-for-safety|здесь]].

---
Помимо отображения сообщений валидации для отдельных свойств, можно отобразить сводку всех сообщений в элементе `<div>` с помощью тег-хелпера Validation Summary:
```html
<div asp-validation-summary="All"></div>
```
Возможно три значения:
- `None` — не отображать;
- `ModelOnly` — отображать ошибки, *не* ассоциированные со каким-либо свойством;
- `All` — отображать ошибки, ассоциированные со свойством, либо с моделью.

Тег-хелпер сводки особенно полезен, если возможны ошибки, не связанные с конкретным свойством. Их можно добавить к состоянию модели, используя пустой ключ:
```csharp
public class ConverModel : PageModel
{
	[BindProperty]
	public InputModel Input { get; set; }
	
	[HttpPost]
	public IActionResult OnPost()
	{
		if(Input.CurrencyFrom == Input.CurrencyTo)
		{
			ModelState.AddModelError(string.Empty, "Cannot convert currency to itself");
		}
		if(!ModelState.IsValid)
		{
			return Page();
		}
		
		//Ошибок нет, сохраняем введенные значения...
		
		return RedirectToPage("Checkout");
	}
}
```
Без тег-хелпера сводки ошибка не была бы отображена, и пользователю не было бы понятно, в чём она заключается.