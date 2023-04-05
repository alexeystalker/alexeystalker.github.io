---
share: true
tags:
 - NET/ASPNETCore/validation
---
# Добавляем FluentValidation в приложение
Чтобы добавить [[library-fluentvalidation|FluentValidation]] в приложение, нужно сделать следующее:
1. Установить NuGet-пакет:
	```bash
	dotnet add package FluentValidation
	```
2. Добавить библиотеку в методе `ConfigureServices`, вызвав метод `AddFluentValidation`. Также можно добавить дополнительные настройки;
3. Зарегистрировать валидаторы в [[di-container|контейнере зависимостей]] (можно использовать любую область действия):
	```csharp
	services.AddScoped<IValidator<CurrencyConverterModelValidator>,
		CurrencyConverterModelValidator>();
	```
	Или же можно разрешить FluentValidation автоматически регистрировать все валидаторы (см. ниже).
	
Также можно настроить библиотеку при добавлении. В следующем примере мы укажем, что нужно автоматически регистрировать все валидаторы, а также отключим поддержку локализации и использование `DataAnnotations`, а также включим проверку вложенных свойств.
```csharp
public void ConfigureServices(IServiceCollection services)
{
	services.AddRazorPages()
		.AddFluentValidation(options => 
		{
			options.RegisterValidatorsFromAssemblyContaining<Startup>();
			options.ImplicitlyValidateChildProperties = true;
			options.LocalizationEnabled = false;
			options.RunDefaultMvcValidationAfterFluentValidationExecutes = false;
		})
}
```