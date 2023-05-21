---
share: true
tags:
 - NET/ASPNETCore/validation
---
# Сравнение FluentValidation и атрибутов DataAnnotations
Возьмем модель из [[ch-20-building-custom-validation-attribute|предыдущего раздела]].
```csharp
public class CurrencyConverterModel
{
	public string CurrencyFrom { get; set; }
	public string CurrencyTo { get; set; }
	public string Quantity { get; set; }
}
```
В [[library-fluentvalidation|FluentValidation]] правила валидации определяются в отдельном классе, на каждую модель по классу, обычно наследуясь от `AbstractValidator<>`. Создадим валидатор, аналогичный правилам из предыдущего раздела. Набор правил для проверки свойства создаётся, вызывая метод `RuleFor()` и добавляя методы правил. Данный способ называется FluentAPI, откуда и название библиотеки.
```csharp
public class CurrencyConverterModelValidator 
	: AbstractValidator<CurrencyConverterModel>
{
	private readonly string[] _allowedValues 
		= new []{ "GBP", "USD", "CAD", "EUR" };
	
	public CurrencyConverterModelValidator()
	{
		RuleFor(x => x.CurrencyFrom)
			.NotEmpty()
			.Length(3)
			.Must(value => _allowedValues.Contains(value))
			.WithMessage("Not a valid currency code");
			
		RuleFor(x => x.CurrencyTo)
			.NotEmpty()
			.Length(3)
			.Must(value => _allowedValues.Contains(value))
			.WithMessage("Not a valid currency code");
			
		RuleFor(x => x.Quantity)
			.NotNull()
			.InclusiveBetween(1, 1000);
	}
}
```
Если правило (в этом примере - для кода валюты) планируется использовать в нескольких валидаторах, есть смысл его вынести, например, в метод расширения.
```csharp
public static class ValidationExtensions
{
	public static IRuleBuilderOptions<T, string> MustbeCurrencyCode<T>(
		this IRuleBuilder<T, string> ruleBuilder)
	{
		return ruleBuilder
			.Must(value => _allowedValues.Contains(value))
			.WithMessage("Not a valid currency code");
	}
	private static readonly string[] _allowedValues 
		= new []{ "GBP", "USD", "CAD", "EUR" };
}
```
Теперь его можно использовать в нашем валидаторе:
```csharp
RuleFor(x => x.CurrencyTo)
	.NotEmpty()
	.Length(3)
	.MustBeCurrencyCode();
```
---
Ещё одно преимущество FluentValidation — возможность внедрять сервисы без применения *Локатора сервисов (Service locator)*. Внедрим `ICurrencyProvider`:
```csharp
public class CurrencyConverterModelValidator 
	: AbstractValidator<CurrencyConverterModel>
{
	public CurrencyConverterModelValidator(ICurrencyProvider provider)
	{
		RuleFor(x => x.CurrencyFrom)
			.NotEmpty()
			.Length(3)
			.Must(value => provider.GetCurrencies().Contains(value))
			.WithMessage("Not a valid currency code");
			
		RuleFor(x => x.CurrencyTo)
			.NotEmpty()
			.Length(3)
			.Must(value => provider.GetCurrencies().Contains(value))
			.WithMessage("Not a valid currency code");
			
		RuleFor(x => x.Quantity)
			.NotNull()
			.InclusiveBetween(1, 1000);
	}
}
```
---
Теперь рассмотрим валидатор, охватывающий несколько свойств. Например, мы хотим проверить, что значение `CurrencyTo` не равно значению `CurrencyFrom`. Используем перегруженный вариант `Must`, который предоставляет и и модель, и проверяемое значение:
```csharp
RuleFor(x => x.CurrencyTo)
	.NotEmpty()
	.Length(3)
	.MustBeCurrencyCode()
	.Must((InputModel model, string currencyTo) => currencyTo != model.CurrencyFrom)
	.WithMessage("Cannot convert currency to itserlf");
```
---
Во FluentValidation много и других полезных функций, облегчающих написание и тестирование валидаторов:
- *валидация сложных типов*;
- *специальные валидаторы свойств*;
- *правила для коллекций* — когда типы содержат коллекции, можно применить валидацию как к коллекции в целом, так и к каждому элементу;
- *наборы правил*;
- *валидация на стороне клиента* — FluentValidation генерирует те же атрибуты, что и `DataAnnotations`, для активации проверки на стороне клиента

