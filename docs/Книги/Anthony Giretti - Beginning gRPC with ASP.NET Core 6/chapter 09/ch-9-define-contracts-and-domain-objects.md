---
share: true
---
# Определяем контракты и доменные объекты
Определим контракты — интерфейсы, связывающие слой доступа к данным (DAL) и слой бизнес-логики, а также контракты для связи слоя бизнес-логики и Razor Pages.
Начнем с контрактов, определяющих взаимодействие с **CountryWiki.DAL**. Слой доступа к данным в данном случае будет обращаться к сервису gRPC для получения и изменения данных. Поэтому определим модели `CountryModel`, `CreateCountryModel`, `CreatedCountryModel` и `UpdateCountryModel`.
```csharp
namespace CountryWiki.Domain.Models;

public record class CountryModel
{
    public int Id { get; init; }
    public string Name { get; init; }
    public string Description { get; init; }
    public string FlagUri { get; init; }
    public string CapitalCity { get; init; }
    public string Anthem { get; init; }
    public IEnumerable<string> Languages { get; init; }
}
```
```csharp
namespace CountryWiki.Domain.Models;

public record class CreateCountryModel
{
    public string Name { get; init; }
    public string Description { get; init; }
    public string FlagUri {get; init; }
    public string CapitalCity { get; init; }
    public string Anthem { get; init; }
    public IEnumerable<int> Languages { get; init; }
}
```
```csharp
namespace CountryWiki.Domain.Models;

public record class CreatedCountryModel
{
    public int Id { get; init; }
    public string Name { get; init; }
}
```
```csharp
namespace CountryWiki.Domain.Models;

public record class UpdateCountryModel
{
    public int Id { get; init; }
    public string Description { get; init; }
}
```
Контракт репозитория определим следующим образом:
```csharp
namespace CountryWiki.Domain.Repositories;

public interface ICountryRepository
{
    IAsyncEnumerable<CreatedCountryModel> CreateAsync(IEnumerable<CreateCountryModel> countriesToCreate);
    Task UpdateAsync(UpdateCountryModel countryToUpdate);
    Task DeleteAsync(int id);
    Task<CountryModel> GetAsync(int id);
    IAsyncEnumerable<CountryModel> GetAllAsync();
}
```
Здесь решено открыть возможность стриминга для слоя бизнес-логики при помощи выноса `IAsyncEnumerable` в контракт. Преимущества этого мы увидим позже. Методы `UpdateAsync` и `DeleteAsync` не будут возвращать значений — вместо этого они будут выбрасывать исключение в случае, если операцию не удалось выполнить.

Теперь определим контракты для работы со слоем бизнес-логики. Cперва определим модель `CountryUploadFileModel` — это будет файл с данными о странах.
```csharp
namespace CountryWiki.Domain.Models;

public record class CountryUploadedFileModel
{
    public string FileName { get; init; }
    public string ContentType { get; init; }
}
```
Также нам понадобится определить интерфейс `ICountryFileUploadValidatorService` для сервиса валидации загруженных файлов, и `ICountryServices` для бизнес-логики CRUD-операций.
```csharp
namespace CountryWiki.Domain.Services;

public interface ICountryFileUploadValidatorService
{
    bool ValidateFile(CountryUploadedFileModel countryUploadedFile);
    Task<IEnumerable<CreateCountryModel>?> ParseFile(Stream content);
}
```
```csharp
namespace CountryWiki.Domain.Services;

public interface ICountryServices
{
    Task CreateAsync(IEnumerable<CreateCountryModel> countriesToCreate);
    Task UpdateAsync(UpdateCountryModel countryToUpdate);
    Task DeleteAsync(int id);
    Task<CountryModel> GetAsync(int id);
    Task<IEnumerable<CountryModel>> GetAllAsync();
}
```
Здесь мы не возвращаем из метода `CreateAsync` значений, так как хотим, чтобы слой бизнес-логики сам обрабатывал результаты операции создания. А фронтенд (то есть проект `CountryWiki.Web`) будет только отображать запрошенную информацию, а не результат выполнения операций — он будет только залогирован. Ошибки будут отображаться в случае их возникновения.
