---
share: true
tags:
 - gRPC/validation
---
# Валидируем сообщения
Очень важдно проверять вводимые пользователем данные, чтобы избежать некорректных с точки зрения бизнес-правил или даже злонамеренно испорченных данных. В данном разделе мы рассмотрим, как организовать проверку (или валидацию) данных.
Не существует каких-то встроенных механизмов или пакетов валидации, в отличие от тех же `DataAnnotations` в ASP.NET. Поэтому автор предлагает воспользоваться разработанным им NuGet-пакетом, основанным на библиотеке [[library-fluentvalidation|FluentValidation]] — [Calzolari.Grpc.AspNetCore.Validation](https://www.nuget.org/packages/Calzolari.Grpc.AspNetCore.Validation). Добавим его к нашему проекту.

Давайте напишем валидатор для сообщения `CountryCreationRequest`, в котором поле `Name` сделаем обязательным, а поле `Description` — не короче 5 символов.
```csharp
using CountryService.Web.gRPC;
using FluentValidation;

namespace CountryService.Web.Validator;

public class CountryCreateRequestValidator: AbstractValidator<CountryCreationRequest>
{
    public CountryCreateRequestValidator()
    {
        RuleFor(request => request.Name).NotEmpty().WithMessage("Name is mandatory.");
        RuleFor(request => request.Description).MinimumLength(5)
            .WithMessage("Description is mandatory and should be longer than 4 characters");
    }
}
```
Добавляем валидатор в файле **Program.cs**:
```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddGrpc(options =>
{
	// прочие опции...
    options.EnableMessageValidation(); //Метод расширения из пакета!
});
builder.Services.AddGrpcValidation(); //Добавляем сервисы валидации
builder.Services.AddValidator<CountryCreateRequestValidator>(); //Добавляем сам валидатор

// прочий код для запуска приложения...

```

Теперь, если мы попытаемся создать объект, но не выполним какое-либо условие (например, напишем поле Description меньше пяти символов) - получим ошибку `InvalidArgument`.
