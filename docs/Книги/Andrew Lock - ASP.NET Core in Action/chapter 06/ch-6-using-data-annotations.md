---
share: true
tags:
 - NET/ASPNETCore/validation
 - NET/DataAnnotations
---
# Использование атрибутов DataAnnotations для валидации
Атрибуты `DataAnnotations` позволяют указать правила, которым должна соответствовать [[binding-model|модель привязки]].
Рассмотрим класс `UserBindingModel` из [[ch-6-binding-complex-types|предыдущего раздела]].
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
Вот некоторые из доступных атрибутов:
- `[CreditCard]` — проверяет, что свойство имеет допустимый формат номера кредитной карты;
- `[EmailAddress]` — проверяет, что свойство имеет допустимый формат адреса электронной почты;
- `[StringLength(max)]` — проверяет, что строка имеет не более максимального количества символов;
- `[MinLength(min)]` — проверяет, что коллекция имеет как минимум минимальное количество элементов;
- `[Phone]` — проверяет, что свойство имеет формат телефонного номера;
- `[Range(min, max)]` — проверяет, что свойство имеет значение от минимального до максимального;
- `[RegularExpression(regex)]` — проверяет, соответствует ли свойство шаблону регулярного выражения;
- `[Url]` — проверяет, что свойство имеет допустимый формат URL;
- `[Required]` — указывает, что свойство не должно иметь значение `null`;
- `[Compare]` — позволяет подтвердить, что два свойства имеют одинаковое значение.

В пространстве имён `System.ComponentModel.DataAnnotations` есть и другие полезные атрибуты. Также можно написать свой атрибут, [[ch-20-building-custom-validation-attribute|унаследовав]] его от `ValidationAttribute`.
