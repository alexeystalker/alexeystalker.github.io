---
share: true
---
# Реализуем слой бизнес-логики
Настало время для реализации слоя бизнес-логики. Сделаем его реализацию простой — мы хотим наблюдать (при помощи логов) за потоковыми данными. Напоминаем, что на уровне бизнес-логики не должно быть известно, являются ли данные потоковыми или нет. Поэтому мы сохраним использование `IAsyncEnumerable` при получении результатов, но не будем распространять его далее на уровне бизнес-логики и выше.
Вот код реализации класса `CountryServices`.
```csharp
namespace CountryWiki.BLL.Services;

public class CountryServices : ICountryServices
{
    private readonly ICountryRepository _countryRepository;
    private readonly ILogger<CountryServices> _logger;

    public CountryServices(ICountryRepository countryRepository, ILogger<CountryServices> logger)
    {
        _countryRepository = countryRepository;
        _logger = logger;
    }

    public async Task CreateAsync(IEnumerable<CreateCountryModel> countriesToCreate)
    {
        await foreach (var createdCountry in _countryRepository.CreateAsync(countriesToCreate))
        {
            _logger.LogDebug($"Country {createdCountry.Name} has been created successfully with Id {createdCountry.Id}");
        }
    }

    public async Task DeleteAsync(int id)
    {
        await _countryRepository.DeleteAsync(id);
        _logger.LogDebug($"Country with Id {id} has been successfully deleted");
    }

    public async Task<IEnumerable<CountryModel>> GetAllAsync()
    {
        var countries = new List<CountryModel>();
        await foreach (var country in _countryRepository.GetAllAsync())
        {
            countries.Add(country);
        }
        return countries;
    }

    public Task<CountryModel> GetAsync(int id)
    {
        return _countryRepository.GetAsync(id);
    }

    public async Task UpdateAsync(UpdateCountryModel countryToUpdate)
    {
        await _countryRepository.UpdateAsync(countryToUpdate);
        _logger.LogDebug($"Country with Id {countryToUpdate.Id} has been successfully updated");
    }
}
```
Также нам нужно добавить реализацию валидатора загружаемых файлов:
```csharp
namespace CountryWiki.BLL.Services;

public class CountryFileUploadValidatorService : 
    ICountryFileUploadValidatorService
{
    public CountryFileUploadValidatorService() { }

    public bool ValidateFile(CountryUploadedFileModel countryUploadedFile)
    {
        return countryUploadedFile.FileName.ToLower().EndsWith(".json") &&
               countryUploadedFile.ContentType == "application/json";
    }

    public async Task<IEnumerable<CreateCountryModel>?> ParseFile(Stream content)
    {
        try
        {
            var parsedCountries = await JsonSerializer
                .DeserializeAsync<IEnumerable<CreateCountryModel>>(
                    content, 
                    new JsonSerializerOptions
                    {
                        PropertyNameCaseInsensitive = true
                    });

            return (parsedCountries ?? Array.Empty<CreateCountryModel>())
                .Any(x => string.IsNullOrEmpty(x.Name) ||
                          string.IsNullOrEmpty(x.Anthem) ||
                          string.IsNullOrEmpty(x.Description) ||
                          string.IsNullOrEmpty(x.FlagUri) ||
                          string.IsNullOrEmpty(x.CapitalCity) ||
                          !x.Languages.Any())
                ? null
                : parsedCountries;
        }
        catch
        {
            return null;
        }
    }
}
```
В валидаторе мы не хотим выбрасывать исключение, если что-то пошло не так, вместо этого просто вернем `null` и покажем пользователю сообщение.