---
share: true
tags:
 - gRPC/unary
 - gRPC/server-streaming
 - gRPC/client-streaming
 - gRPC/duplex-streaming
 - NET/ASPNETCore
---
# Пишем, конфигурируем и доставляем сервисы gRPC
Созданные в [[ch-5-create-and-compile-protobuf-files|прошлом разделе]] заглушки gRPC представляют собой абстрактные классы. Более того, если посмотреть в содержимое файла **CountryGrpc.cs**, можно увидеть, что методы, реализующие функции сервиса, возвращают [[ch-3-grpc-status|gRPC-статус]] `Unimplemented`. Чтобы всё заработало, нужно написать код, переопределяющий выполнение этих методов.
Создадим класс под названием `CountryGrpcService` в папке **Services**. Сменим объявление пространства имён на file-scoped и добавим статический юзинг `using static CountryService.Web.gRPC.CountryService;` для удобства. Унаследуем наш класс от `CountryServiceBase`. Должно получиться примерно следующее:
```csharp
using static CountryService.Web.gRPC.CountryService;

namespace CountryService.Web.Services;

public class CountryGrpcService : CountryServiceBase
{
}
```
Теперь переопределим методы нашего сервиса. Уберём указания на пространства имён типа `global::%что-то%`,  при необходимости добавляя их в юзинги в шапке файла.
В итоге мы должны получить такой файл:
```csharp
using CountryService.Web.gRPC;
using Google.Protobuf.WellKnownTypes;
using Grpc.Core;
using static CountryService.Web.gRPC.CountryService;

namespace CountryService.Web.Services;

public class CountryGrpcService : CountryServiceBase
{
    public override Task GetAll(Empty request, IServerStreamWriter<CountryReply> responseStream, ServerCallContext context)
    {
        throw new RpcException(new Status(StatusCode.Unimplemented, ""));
    }
    public override Task<CountryReply> Get(CountryIdRequest request, ServerCallContext context)
    {
        throw new RpcException(new Status(StatusCode.Unimplemented, ""));
    }
    public override Task<Empty> Delete(IAsyncStreamReader<CountryIdRequest> requestStream, ServerCallContext context)
    {
        throw new RpcException(new Status(StatusCode.Unimplemented, ""));
    }
    public override Task<Empty> Update(CountryUpdateRequest request, ServerCallContext context)
    {
        throw new RpcException(new Status(StatusCode.Unimplemented, ""));
    }
    public override Task Create(IAsyncStreamReader<CountryCreationRequest> requestStream, IServerStreamWriter<CountryCreationReply> responseStream, ServerCallContext context)
    {
        throw new RpcException(new Status(StatusCode.Unimplemented, ""));
    }
}
```
Обратим внимание, что в случае, когда функция gRPC получает или возвращает сообщение не через поток - метод получает объект нужного типа (или `Empty`) в качестве параметра, а возвращает в качестве `Task<T>`, как и полагается для асинхронных методов. В случае же с потоками, клиентский поток передаётся в виде `IAsyncStreamReader<T>`, а серверный в виде `IServerStreamWriter<T>`. Параметр `ServerCallContext` — контекст запроса, аналог `HttpContext`. %% добавить ссылку на главу 14 %%. 
Чтобы наш сервис был доступен извне, нужно сделать кое-то ещё. Добавим [[endpoint|конечную точку]] в файле **Program.cs**
```csharp
using CountryService.Web.Services;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddGrpc(); //Добавляем сервисы в контейнер зависимостей

var app = builder.Build();

app.MapGrpcService<CountryGrpcService>(); //Добавляем наш сервис как конечную точку.

app.MapGet("/", () => "Communication with gRPC endpoints must be made through a gRPC client. To learn how to create a client, visit: https://go.microsoft.com/fwlink/?linkid=2086909");

app.Run();

```
Теперь нам надо реализовать собственно CRUD операции. Для этой главы реализуем простейший сервис, хранящий данные в памяти. %% Добавить ссылку на главу 9, в которой показано как сделать приложение с доступом к настоящей БД %%
Назовём его `CountryManagementService`.
```csharp
using CountryService.Web.gRPC;
using Google.Protobuf.WellKnownTypes;

namespace CountryService.Web
{
    public class CountryManagementService
    {
        private readonly List<CountryReply> _countries = new();

        public CountryManagementService()
        {
            _countries.Add(new CountryReply
            {
                Id = 1,
                Name = "Canada",
                Description = "Canada has at least 32 000 lakes",
                CreateDate = Timestamp.FromDateTime(DateTime.SpecifyKind(new DateTime(2021, 1, 2), DateTimeKind.Utc))
            });
            _countries.Add(new CountryReply
            {
                Id = 2,
                Name = "USA",
                Description = "Yellowstone has 300 to 500 geysers",
                CreateDate = Timestamp.FromDateTime(DateTime.SpecifyKind(new DateTime(2021, 1, 2), DateTimeKind.Utc))
            });
            _countries.Add(new CountryReply
            {
                Id = 3,
                Name = "Mexico",
                Description = "Mexico is crossed by Sierra Madre Oriental and Sierra Madre Occidental mountains",
                CreateDate = Timestamp.FromDateTime(DateTime.SpecifyKind(new DateTime(2021, 1, 2), DateTimeKind.Utc))
            });
        }

        public  Task<IEnumerable<CountryReply>> GetAllAsync()
        {
            return Task.FromResult(_countries.AsEnumerable());
        }

        public Task<CountryReply?> GetAsync(CountryIdRequest idRequest)
        {
            return Task.FromResult(_countries.FirstOrDefault(c => c.Id == idRequest.Id));
        }

        public Task DeleteAsync(IEnumerable<CountryIdRequest> idRequests)
        {
            var ids = idRequests.Select(r => r.Id).ToHashSet();
            _countries.RemoveAll(c => ids.Contains(c.Id));
            return Task.CompletedTask;
        }

        public Task UpdateAsync(CountryUpdateRequest updateRequest)
        {
            var countryToUpdate = _countries.FirstOrDefault(c => c.Id == updateRequest.Id);
            if (countryToUpdate != null)
            {
                countryToUpdate.Description = updateRequest.Description;
                countryToUpdate.UpdateDate = updateRequest.UpdateDate;
            }
            return Task.CompletedTask;
        }

        public Task<IEnumerable<CountryCreationReply>> CreateAsync(IEnumerable<CountryCreationRequest> creationRequests)
        {
            var countriesCount = _countries.Count;
            var newCountries = creationRequests
                .Where(cr => _countries.All(c => c.Name != cr.Name))
                .Select(cr => new CountryReply
                {
                    Id = ++countriesCount,
                    Name = cr.Name,
                    Description = cr.Description,
                    Flag = cr.Flag,
                    CreateDate = Timestamp.FromDateTime(DateTime.SpecifyKind(DateTime.Today, DateTimeKind.Utc))
                }).ToList();
            _countries.AddRange(newCountries);
            var result = newCountries.Select(nc => new CountryCreationReply
            {
                Id = nc.Id,
                Name = nc.Name
            });
            return Task.FromResult(result);
        }
    }
}

```
Зарегистрируем класс в [[di-container|контейнере зависимостей]] в файле **Program.cs**:
```csharp
builder.Services.AddSingleton<CountryManagementService>();
```
Теперь можно внедрить `CountryManagementService` в конструкторе `CountryGrpcService` и набросать реализацию CRUD-операций
```csharp
using CountryService.Web.gRPC;
using Google.Protobuf.WellKnownTypes;
using Grpc.Core;
using static CountryService.Web.gRPC.CountryService;

namespace CountryService.Web.Services;

public class CountryGrpcService : CountryServiceBase
{
    private readonly CountryManagementService _countryManagementService;

    public CountryGrpcService(CountryManagementService countryManagementService)
    {
        _countryManagementService = countryManagementService;
    }

    public override async Task GetAll(Empty request, IServerStreamWriter<CountryReply> responseStream, ServerCallContext context)
    {
        //Стримим все найденные страны клиенту
        var replies = await _countryManagementService.GetAllAsync();
        foreach (var countryReply in replies)
        {
            await responseStream.WriteAsync(countryReply);
        }
    }

    public override async Task<CountryReply> Get(CountryIdRequest request, ServerCallContext context)
    {
        //Нам может вернуться null, если передан несуществующий Id, вернем NotFound в этом случае
        var result = await _countryManagementService.GetAsync(request);
        return result ?? throw new RpcException(new Status(StatusCode.NotFound, $"No country with id {request.Id}"));
    }

    public override async Task<Empty> Delete(IAsyncStreamReader<CountryIdRequest> requestStream, ServerCallContext context)
    {
        //Сперва загрузим все запросы на удаление
        var requestsList = new List<CountryIdRequest>();
        await foreach (var idRequest in requestStream.ReadAllAsync())
        {
            requestsList.Add(idRequest);
        }
        //Теперь удалим всё разом
        await _countryManagementService.DeleteAsync(requestsList);

        return new Empty();
    }

    public override async Task<Empty> Update(CountryUpdateRequest request, ServerCallContext context)
    {
        await _countryManagementService.UpdateAsync(request);
        return new Empty();
    }

    public override async Task Create(IAsyncStreamReader<CountryCreationRequest> requestStream, IServerStreamWriter<CountryCreationReply> responseStream, ServerCallContext context)
    {
        //Сперва загрузим все запросы на создание
        var requestsList = new List<CountryCreationRequest>();
        await foreach (var createRequest in requestStream.ReadAllAsync())
        {
            requestsList.Add(createRequest);
        }
        //сохраним всё сразу
        var createdCountries = await _countryManagementService.CreateAsync(requestsList);
        foreach (var createdCountry in createdCountries)
        {
            await responseStream.WriteAsync(createdCountry);
        }

    }
}
```
Помимо этого можно добавить несколько опций и тем самым изменить или улучшить поведение сервиса gRPC. В [[ch-3-grpc-channel|главе 3]] было показано, как добавить опции к каналу, используемому в клиенте. Рассмотрим, как сделать то же самое на стороне сервера. Вот какие настройки можно изменить глобально или для каждого сервиса отдельно.
- `MaxSendMessageSize` — максимальный размер сообщения, отправляемого с сервера. Если значение не задано, лимит отсутствует. Если значение задано, и размер сообщения превысит его — будет выброшено исключение `RpcException`;
- `MaxReceivedMessageSize` — максимальный размер сообщения, отправляемого на сервер. По умолчанию равно 4 Мб. Чтобы снять лимит, нужно присвоить `null`. При превышении лимита выбрасывается исключение `RpcException`;
- `CompressionProviders` — коллекция поставщиков (провайдеров) сжатия. Если ничего не было задано, провайдером по умолчанию будет Gzip. Можно настроить сжатие Gzip и/или добавить свой провайдер;
- `ResponseCompressionAlgorithm` — строковое обозначение алгоритма, использованного при сжатии. Если не задано, будет использован первый подходящий под заголовок `grpc-accept-encoding` из текущей коллекции. Алгоритмом по умолчанию является Gzip. Его не нужно добавлять в `ComperssionProviders`.
- `ResponseCompressionLevel` — уровень сжатия, передаваемый провайдеру сжатия. Провайдер использует уровень сжатия по умолчанию в случае, если уровень не задан этим свойством.
#### [[ch-5-add-compression-provider|Добавим провайдер сжатия]]
> [!tip] Совет
> Несмотря на то, что `MaxReceiveMessageSize` и `MaxSendMessageSize` необязательны, настоятельно рекомендуется использовать их для ограничения потребляемых ресурсов.

Кроме этого, есть ещё настройки `EnableDetailedErrors`, `Interceptors` и `IgnoreUnknownServices`, но их мы разберем [[ch-5-manage-errors-handle-responses-and-perform-logging|позже]].