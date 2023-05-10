---
share: true
tags:
 - NET/ASPNETCore/validation
---
# Создание специального атрибута валидации
В [[ch-6-validating-on-server-for-safety|главе 6]] мы рассмотрели, как использовать встроенные атрибуты `DataAnnotations` в своих [[binding-model|моделях привязки]]. Но как быть, если эти атрибуты не отвечают нашим требованиям? Например, необходимо проверить, что переданное свойство содержит не просто строку из трёх символов, но строку из определённого набора (как это нужно, например, для приложения конвертации валют).
Рассмотрим, как создать специальный атрибут, унаследованный от `ValidationAttribute`.
```csharp
public class CurrencyCodeAttribute: ValidationAttribute
{
	private readonly string[] _allowedCodes;
	public CurrencyCodeAttribute(params string[] allowedCodes)
	{
		_allowedCodes = allowedCodes;
	}
	
	protected override ValidationResult IsValid(
		object value, ValidationContext context)
	{
		var code = value as string;
		if(code == null || !_allowedCodes.Contains(code))
		{
			return new ValidationResult("Not a valid currency code");
		}
		return ValidationResult.Success;
	}
}
```
Валидация выполняется в [[ch-13-action-filters|конвейере фильтров действий]]. Фреймворк валидации вызывает метод `IsValid()` для каждого экземпляра `ValidationAttribute` проверяемого свойства модели и передаёт `value` — значение проверяемого свойства и `ValidationContext` к каждому атрибуту по очереди. Объект контекста содержит детали, которые можно использовать для валидации свойства.
Особо отметим свойство `ObjectInstance`. Его можно использовать для доступа к проверяемой модели верхнего уровня при проверке вложенного свойства. Это может быть полезно, если допустимость свойства зависит от значения другого свойства модели. 
Например, если мы проверяем свойство `CurrencyFrom` из модели `CurrencyConverterModel`, объект верхнего уровня можно получить так:
```csharp
var model = validationContext.ObjectInstance as CurrencyConverterModel;
```
> [!Note] Примечание
> Очевидно, использование `ObjectInstance` снижает переносимость атрибута валидации. В описанном случае использовать атрибут можно только в приложении, которое определяет `CurrencyConverterModel`.

Если для валидации необходимо использовать дополнительные сервисы (например, получить список допустимых валют), необходимо использовать *локатор сервисов*.
Перепишем код атрибута валидации, чтобы получить список валют из гипотетического сервиса `ICurrencyProvider`.
```csharp
public class CurrencyCodeAttribute: ValidationAttribute
{

	protected override ValidationResult IsValid(
		object value, ValidationContext context)
	{
		var provider = context.GetService<ICurrencyProvider>();
		var allowedCodes = provider.GetCurrencies();
		
		var code = value as string;
		if(code == null || !allowedCodes.Contains(code))
		{
			return new ValidationResult("Not a valid currency code");
		}
		return ValidationResult.Success;
	}
}
```