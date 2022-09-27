---
share: true
tags:
 - NET/Razor/tag-helper
---
# Тег-хелпер метки (label)
Каждое поле `<input>` должно иметь ассоциированный тег `<label>`. Можно либо самому задавать имя поля и заполнять атрибут `for`, но есть тег-хелпер, который всё сделает сам.
Тег-хелпер метки используется для задания текста и атрибута `for` элемента `<label>` на основе свойств в `PageModel`:
```html
<label asp-for="FirstName"></label>
```
Тег-хелпер использует атрибут `[Display]` для отображения. Если у свойства нет атрибута `[Display]`, будет использовано имя свойства.
Предположим, модель такая:
```csharp
public class UserModel
{
	[Display(Name = "Your name")]
	public string FirstName { get; set; }
	public string Email { get;set; }
}
```
Используем теги:
```html
<label asp-for="FirstName"></label>
<label asp-for="Email"></label>
```
Получим в результате:
```html
<label for="FirstName">Your name</label>
<label for="Email">Email</label>
```
Атрибут `for` важен для доступности; он определяет идентификатор элемента, к которому относится метка, что может быть важно, например, при использовании программы для чтения с экрана.
Также можно ссылаться на вложенные свойства. Например, пусть есть свойство `Input`:
```csharp
public class CheckoutModel : PageModel
{
	[BindProperty]
	public UserModel Input { get; set; }
}
```
Тогда сослаться на свойства `FirstName` и `Email` можно так:
```html
<label asp-for="Input.FirsName"></label>
<label asp-for="Input.Email"></label>
```
Тег-хелпер не переопределяет значение, введенное явно.
То есть
```html
<label asp-for="Email">Please enter your email</label>
```
сгенерирует код
```html
<label for="Email">Please enter your email</label>
```
