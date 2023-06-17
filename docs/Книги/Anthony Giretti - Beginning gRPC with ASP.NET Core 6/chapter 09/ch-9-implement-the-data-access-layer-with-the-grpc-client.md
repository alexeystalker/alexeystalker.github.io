---
share: true
tags:
 - gRPC/client
---
# Реализуем слой доступа к данным при помощи клиента gRPC
Теперь нам нужно создать слой доступа к данным. Так как доступ к данным будет осуществляться при помощи gRPC, нам нужно добавить клиента для нашего gRPC сервиса. Мы рассмотрели процедуру в [[ch-7-create-a-grpc-client|главе 7]]. Добавим файл **country.proto** к нашему проекту **CountryWiki.DAL**, и не забудем поменять значение опции `csharp_namespace` на подходящее нашему проекту — `CountryWiki.DAL.v1`.
После добавления файла `.proto` и компиляции будет создан, среди прочих, класс `CountryServiceClient`. Теперь, используя его, можно реализовать интерфейс `ICountryRepository`:
```csharp
namespace CountryWiki.DAL.Repositories;

public class CountryRepository : ICountryRepository
{
    private readonly CountryServiceClient _countryServiceClient;

    public CountryRepository(CountryServiceClient countryServiceClient)
    {
        _countryServiceClient = countryServiceClient;
    }
    public async IAsyncEnumerable<CreatedCountryModel> CreateAsync(IEnumerable<CreateCountryModel> countriesToCreate)
    {
        using var bidirectionalStreamingCall = _countryServiceClient.Create();
        foreach (var countryToCreate in countriesToCreate)
        {
            var countryToCreateRequest = new CountryCreationRequest
            {
                Name = countryToCreate.Name,
                Description = countryToCreate.Description,
                Anthem = countryToCreate.Anthem,
                CapitalCity = countryToCreate.CapitalCity,
                FlagUri = countryToCreate.FlagUri
            };
            countryToCreateRequest.Languages.AddRange(countryToCreate.Languages);
            await bidirectionalStreamingCall.RequestStream.WriteAsync(countryToCreateRequest);
        }
        // отправляем уведомление о том, что мы закончили
        await bidirectionalStreamingCall.RequestStream.CompleteAsync();

        // Читаем
        while (await bidirectionalStreamingCall.ResponseStream.MoveNext(CancellationToken.None))
        {
            var country = bidirectionalStreamingCall.ResponseStream.Current;
            yield return new CreatedCountryModel
            {
                Id = country.Id,
                Name = country.Name
            };
        }
    }
    
    public async Task DeleteAsync(int id) => 
        await _countryServiceClient.DeleteAsync(new CountryIdRequest {Id = id});

    public async IAsyncEnumerable<CountryModel> GetAllAsync()
    {
        using var serverStreamingCall = _countryServiceClient.GetAll(new Empty());
        while (await serverStreamingCall.ResponseStream.MoveNext(CancellationToken.None))
        {
            yield return serverStreamingCall.ResponseStream.Current.ToDomain();
        }
    }
    
    public async Task<CountryModel> GetAsync(int id) => 
        (await _countryServiceClient.GetAsync(new CountryIdRequest {Id = id})).ToDomain();

    public async Task UpdateAsync(UpdateCountryModel countryToUpdate) =>
        await _countryServiceClient.UpdateAsync(new CountryUpdateRequest
        {
            Id = countryToUpdate.Id,
            Description = countryToUpdate.Description,
        });
}
```
Методы `CreateAsync` и `GetAllAsync` возвращают `IAsyncEnumerable`. Мы воспользуемся этим, когда будем реализовывать слой бизнес-логики, и добавим дополнительную логику — логирование — для каждого полученного объекта.

Как и в случае с **CountryService**, метод `ToDomain` — extension-метод:
```csharp
namespace CountryWiki.DAL.Mappers;

public static class CountryModelMappers
{
    public static CountryModel ToDomain(this CountryReply countryReply) =>
            new CountryModel
            {
                Id = countryReply.Id,
                Name = countryReply.Name,
                Description = countryReply.Description,
                Anthem = countryReply.Anthem,
                FlagUri = countryReply.FlagUri,
                CapitalCity = countryReply.CapitalCity,
                Languages = countryReply.Languages
            };
}
```
