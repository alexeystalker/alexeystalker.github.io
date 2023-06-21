---
share: true
---
# Пишем бизнес-логику в слое CountryService.BLL
Теперь напишем класс `CountryServices`. Этот класс реализует CRUD-операции над доменными объектами, а также некоторую логику, являющуюся посредником между слоем ASP.NET Core, доступным извне, и сырыми данными, получаемыми из источника данных.
Сперва напишем контракт сервиса, и разместим его в `CountryService.Domain`.
```csharp
namespace CountryService.Domain.Services;

public interface ICountryServices
{
    Task<int> CreateAsync(CreateCountryModel countryToCreate);
    Task<bool> UpdateAsync(UpdateCountryModel countryToUpdate);
    Task<bool> DeleteAsync(int id);
    Task<CountryModel> GetAsync(int id);
    Task<IEnumerable<CountryModel>> GetAllAsync();
}
```
А реализацию поместим в `CountryService.BLL`
```csharp
namespace CountryService.BLL.Services;

public class CountryServices : ICountryServices
{
    private readonly ICountryRepository _countryRepository;

    public CountryServices(ICountryRepository countryRepository)
    {
        _countryRepository = countryRepository;
    }

    public Task<int> CreateAsync(CreateCountryModel countryToCreate) =>
        _countryRepository.CreateAsync(countryToCreate);

    public async Task<bool> UpdateAsync(UpdateCountryModel countryToUpdate) => 
        await _countryRepository.UpdateAsync(countryToUpdate) > 0;

    public async Task<bool> DeleteAsync(int id) => 
        await _countryRepository.DeleteAsync(id) > 0;

    public  Task<CountryModel> GetAsync(int id) => 
        _countryRepository.GetAsync(id);

    public async Task<IEnumerable<CountryModel>> GetAllAsync() => 
        await _countryRepository.GetAllAsync();
}
```
Наша реализация методов `UpdateAsync()` и `DeleteAsync()` скрывает количество измененных строк в БД (которые возвращаются методами репозитория), и возвращает только булево значение успешности операции.
