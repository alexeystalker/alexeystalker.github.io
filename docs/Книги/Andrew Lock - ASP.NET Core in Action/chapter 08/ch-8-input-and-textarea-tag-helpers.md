---
share: true
tags:
 - NET/Razor/tag-helper
---
# Тег-хелперы input и textarea
Существует много разнообразных типов `<input>`, зависящих от типа данных и целесообразности. Тег-хелпер `<input>` использует тип свойства и примененные атрибуты `DataAnnotations` для задания типа `<input>`. Также `DataAnnotations` используются для формирования атрибутов валидации `data-val-*`.
Пусть есть модель с атрибутированным свойством
```csharp
public class UserModel
{
	[EmailAddress]
	[Required]
	public string Email { get;set; }
}
```
добавляем элемент `<input>`:
```html
<input asp-for="Input.Email" />
```
Так как есть атрибут `[EmailAddress]`, будет сгенерирован `<input>` с типом `"email"`:
```html
<input type="email" id="Input_Email" name="Input.Email" value="test@example.com" data-val="true" data-val-email="The Email Address field is not a valid e-mail address." data-val-required="The Email Address field is required." />
```
Видно, что атрибуты `id` и `name` сгенерированы из имени свойства. Также задано начальное значение поля, равное значению, хранящемуся в свойстве.
Набор атрибутов `data-val-*` предназначен для клиентских библиотек JavaScript, обеспечивающих валидацию на стороне клиента.
Тип элемента `<input>` выбирается, как уже было сказано, на основе типа свойства и атрибутов `DataAnnotations`. Вот таблица сопоставления распространенных типов:

|Тип данных|Как он указан|Тип `<input>`|
|---|---|---|
|`byte`, `int`, `short`, `long`, `uint`|Тип свойства|`Number`|
|`decimal`, `double`, `float`|Тип свойства|`Text`|
|`bool`|Тип свойства|`Checkbox`|
|`string`|Тип свойства, атрибут `[DataType(DataType.Text)]`|`Text`|
|`HiddenInput`|Атрибут `[HiddenInput]`|`Hidden`|
|`Password`|Атрибут `[Password]`|`Password`|
|`Phone`|Атрибут `[Phone]`|`Tel`|
|`EmailAddress`|Атрибут `[EmailAddress]`|`Email`|
|`Url`|Атрибут `[Url]`|`url`|
|`Date`|Тип свойства `DateTime`, атрибут `[DataType(DataType.Date)]`|`date`|

Также для настройки отображения данных можно использовать атрибут `asp-format`. За кулисами он использует метод `string.Format()` для значения свойства, передавая строку формата.
Например, мы хотим, чтобы свойство типа `decimal` отображалось с тремя десятичными знаками:
```html
<input asp-for="Dec" asp-format="{0:0.000}" />
```
пусть Dec=1.2, тогда получим следующий HTML:
```html
<input type="text" id="Dec" name="Dec" value="1.200" />
```
---
Помимо тег-хелпера `<input>` существует тег-хелпер `<textarea>`:
```html
<textarea asp-for="Bigtextvalue"></textarea>
```
HTML-код:
```html
<textarea data-val="true" id="Bigtextvalue" name="bigtextvalue" data-val-length="Maximum length 200." data-val-length-max="200" data-val-required="The Multiline field is required." >This is some text, I'm going to display it in a text area</textarea>
```