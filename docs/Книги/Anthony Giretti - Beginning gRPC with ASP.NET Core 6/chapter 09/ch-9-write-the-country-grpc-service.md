---
share: true
tags:
 - gRPC
---
# Пишем сервис gRPC
Настало время написать сервис gRPC. Для этого возьмём наш файл country.proto и адаптируем его под реалии этой главы: добавим поля `Anthem` и `CapitalCity`, изменим поле для флага на `FlagUri`, уберем поля с датами и поменяем значения `package` и `csharp_namespace`. 
> [!Note] Примечание
> В этом разделе мы не будем возвращаться к тому, как добавить файл `.proto` к проекту. Прочитать об этом можно [[ch-5-creating-an-asp-net-core-grpc-application|в главе 5]].

Вот код файла `protobuf`.
```protobuf
syntax = "proto3";

option csharp_namespace = "CountryService.gRPC.v1";

package CountryService.v1;

import "google/protobuf/empty.proto";
import "google/protobuf/timestamp.proto";

service CountryService {
	rpc GetAll(google.protobuf.Empty) returns (stream CountryReply) {}
	rpc Get(CountryIdRequest) returns (CountryReply) {}
	rpc Update(CountryUpdateRequest) returns (google.protobuf.Empty) {}
	rpc Delete(CountryIdRequest) returns (google.protobuf.Empty) {}
	rpc Create(stream CountryCreationRequest) returns (stream CountryCreationReply) {}
}

message CountryReply {
	int32 Id = 1;
	string Name = 2;
	string Description = 3;
	string FlagUri = 4;
	string Anthem = 5;
	string CapitalCity = 6;
	repeated string Languages = 7;
}

message CountryIdRequest {
	int32 Id = 1;
}

message CountryUpdateRequest {
	int32 Id = 1;
	string Description = 2;
}

message CountryCreationRequest {
	string Name = 1;
	string Description = 2;
	string FlagUri = 3;
	string Anthem = 4;
	string CapitalCity = 5;
	repeated int32 Languages = 7;
}

message CountryCreationReply {
	int32 Id = 1;
	string Name = 2;
}
```
А вот реализация сервиса:
```csharp
namespace CountryService.gRPC.Services;

public class CountryGrpcService : CountryServiceBase
{
    private readonly ICountryServices _countryService;

    public CountryGrpcService(ICountryServices countryService)
    {
        _countryService = countryService;
    }

    public override async Task GetAll(Empty request, IServerStreamWriter<CountryReply> responseStream, ServerCallContext context)
    {
        var lst = await _countryService.GetAllAsync();
        foreach (var country in lst)
        {
            await responseStream.WriteAsync(country.ToReply());
        }

        await Task.CompletedTask;
    }

    public override async Task<CountryReply> Get(CountryIdRequest request, ServerCallContext context)
    {
        var country = await _countryService.GetAsync(request.Id);
        if (country == null)
            throw new RpcException(new Status(StatusCode.NotFound, $"Country with Id {request.Id} hasn't been found"));

        return country.ToReply();
    }

    public override async Task<Empty> Update(CountryUpdateRequest request, ServerCallContext context)
    {
        var updateSucceed = await _countryService.UpdateAsync(new UpdateCountryModel
        {
            Id = request.Id,
            Description = request.Description,
            UpdateDate = DateTime.UtcNow
        });
        if (!updateSucceed)
            throw new RpcException(new Status(StatusCode.NotFound,
                $"Country with Id {request.Id} hasn't been updated, it have probably been deleted"));

        return new Empty();
    }

    public override async Task<Empty> Delete(CountryIdRequest request, ServerCallContext context)
    {
        var deleteSucceed = await _countryService.DeleteAsync(request.Id);
        if (!deleteSucceed)
            throw new RpcException(new Status(StatusCode.NotFound,
                $"Country with Id {request.Id} hasn't been deleted, it have probably been deleted"));

        return new Empty();
    }

    public override async Task Create(IAsyncStreamReader<CountryCreationRequest> requestStream, 
        IServerStreamWriter<CountryCreationReply> responseStream, ServerCallContext context)
    {
        await foreach (var countryToCreate in requestStream.ReadAllAsync())
        {
            var createdCountryId = await _countryService.CreateAsync(new CreateCountryModel
            {
                Name = countryToCreate.Name,
                Description = countryToCreate.Description,
                Anthem = countryToCreate.Anthem,
                CapitalCity = countryToCreate.CapitalCity,
                FlagUri = countryToCreate.FlagUri,
                Languages = countryToCreate.Languages,
                CreateDate = DateTime.UtcNow
            });
            await responseStream.WriteAsync(new CountryCreationReply
            {
                Id = createdCountryId,
                Name = countryToCreate.Name
            });
        }
    }
}
```
Метод `ToReply()` — это метод расширения, реализованный в отдельном файле:
```csharp
namespace CountryService.gRPC.Mappers;

public static class CountryReplyMapper
{
    public static CountryReply ToReply(this CountryModel country)
    {
        var countryReply = new CountryReply
        {
            Id = country.Id,
            Name = country.Name,
            Description = country.Description,
            Anthem = country.Anthem,
            CapitalCity = country.CapitalCity,
            FlagUri = country.FlagUri
        };
        countryReply.Languages.AddRange(country.Languages);
        return countryReply;
    }
}
```
Далее, вынесем строку подключения к БД в файл **appsettings.json**:
```json
  "ConnectionStrings": {
    "CountryService": "Server=(LocalDB)\\MSSQLLocalDB;Database=CountryService;Integrated Security=True;MultipleActiveResultSets=True"
  }
```
> [!warning] Внимание!
> В реальных приложениях строки подключения содержат чувствительную информацию, необходимую для подключения. Поэтому хранить её лучше в хранилище секретов. Об этом подробнее рассказано [[ch-11-storing-config-secrets-safely|здесь]].

Теперь настроим приложение, добавив [[ch-5-add-compression-provider|провайдер сжатия]], [[ch-5-interceptors|перехватчики]], [[ch-5-using-grpcurl|рефлексию gRPC]], а также [[ch-6-expose-the-versions-of-your-protobuf-with-asp-net-core-minimal-apis|эндпоинты для предоставления версий Protobuf]], получим следующий файл **Program.cs**:
```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddGrpc(options =>
{
    options.EnableDetailedErrors = true;
    options.IgnoreUnknownServices = true;
    options.MaxReceiveMessageSize = 6291456; // 6 Mb
    options.MaxSendMessageSize = 6291456; // 6 Mb
    options.CompressionProviders = new List<ICompressionProvider>
    {
        new BrotliCompressionProvider() // br
    };
    options.ResponseCompressionAlgorithm = "br";
    options.ResponseCompressionLevel = CompressionLevel.Optimal;
    options.Interceptors.Add<ExceptionInterceptor>();
});
builder.Services.AddGrpcReflection();
builder.Services.AddScoped<ICountryRepository, CountryRepository>();
builder.Services.AddScoped<ICountryServices, CountryServices>();
builder.Services.AddDbContext<CountryContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("CountryService")));
builder.Services.AddSingleton<ProtoService>();
var app = builder.Build();

app.MapGrpcReflectionService();
app.MapGrpcService<CountryGrpcService>();

app.MapGet("/protos", (ProtoService protoService) => Results.Ok(protoService.GetAll()));
app.MapGet("/protos/v{version:int}/{protoName}", (ProtoService protoService, int version, string protoName) =>
{
    var filePath = protoService.Get(version, protoName);
    return filePath != null ? Results.File(filePath) : Results.NotFound();
});
app.MapGet("/protos/v{version:int}/{protoName}/view",
    async (ProtoService protoService, int version, string protoName) =>
{
    var text = await protoService.ViewAsync(version, protoName);
    return !string.IsNullOrEmpty(text) ? Results.Text(text) : Results.NotFound();
});

app.Run();
```