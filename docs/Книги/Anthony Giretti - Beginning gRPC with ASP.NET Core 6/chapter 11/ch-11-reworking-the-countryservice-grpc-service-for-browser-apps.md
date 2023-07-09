---
share: true
tags:
 - gRPC/gRPC-web
---
# Переделываем gRPC сервис CountryService для браузерных приложений
Как уже говорилось, браузеры не поддерживают [[grpc-client-streaming-call|клиентские потоки]] и [[grpc-duplex-streaming-call|двунаправленные потоки]]. Поэтому будет необходимо передавать клиентам одни и те же данные независимо от того, используется ли gRPC-web в браузере или нет. 
Сперва рассмотрим архитектуру приложения с сервисом gRPC-web:
![[Pasted image 20230709143541.png]]
Сравнивая этот вариант с [[ch-9-import-and-display-data-with-asp-net-core-razor-pages-hosted-services-and-grpc|вариантом из 9 главы]], можно отметить такие изменения:
- файл с данными теперь парсится и валидируется на стороне фронтенда;
- отсутствует фоновое задание (background task), позволяющее загружать страны в потоковом режиме. Вместо этого есть аналоги на фронтенде: Web Workers или .NET Tasks в случае Blazor WebAssembly.

Мы переместили ответственность за валидацию и парсинг загруженного файла на фронтенд, оставив сервису gRPC только CRUD-операции, что позволит перенести нагрузку на фронтенд и сохранить стабильность сервера.

Теперь нам нужно исправить файлы Protobuf с учётом того, что мы больше не можем использовать двухсторонние потоки. Заведем два различных файла:
- `country.proto` — файл для других серверных приложений или небраузерных клиентов;
- `country.browser.proto` — новый файл для браузерных клиентов.

Так как мы будем использовать одинаковые сообщения, вынесем их определения в отдельный файл `country.shared.proto`, который подключим к обоим нашим файлам. Также добавим новое сообщение `CountriesCreationRequest` со списком `CountryCreationRequest`. Это необходимо, так как мы больше не можем отправлять поток с клиента, и вынуждены отправлять в RPC-метод `Create()` весь список стран целиком.
Вот как выглядит `country.shared.proto`:
```protobuf
syntax = "proto3";

option csharp_namespace = "CountryService.gRPC.Protos.v1";

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

message CountriesCreationRequest {
	repeated CountryCreationRequest Countries = 1;
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
Теперь составим файл `country.browser.proto`:
```protobuf
syntax = "proto3";

option csharp_namespace = "CountryService.gRPC.Browser.v1";

package CountryService.Browser.v1;

import "google/protobuf/empty.proto";
import "Protos/v1/country.shared.proto";

service CountryServiceBrowser {
	rpc GetAll(google.protobuf.Empty) returns (stream CountryReply) {}
	rpc Get(CountryIdRequest) returns (CountryReply) {}
	rpc Update(CountryUpdateRequest) returns (google.protobuf.Empty) {}
	rpc Delete(CountryIdRequest) returns (google.protobuf.Empty) {}
	rpc Create(CountriesCreationRequest) returns (stream CountryCreationReply) {}
}
```
И изменим файл `country.proto`:
```protobuf
syntax = "proto3";

option csharp_namespace = "CountryService.gRPC.Browser.v1";

package CountryService.Browser.v1;

import "google/protobuf/empty.proto";
import "Protos/v1/country.shared.proto";

service CountryServiceBrowser {
	rpc GetAll(google.protobuf.Empty) returns (stream CountryReply) {}
	rpc Get(CountryIdRequest) returns (CountryReply) {}
	rpc Update(CountryUpdateRequest) returns (google.protobuf.Empty) {}
	rpc Delete(CountryIdRequest) returns (google.protobuf.Empty) {}
	rpc Create(CountriesCreationRequest) returns (stream CountryCreationReply) {}
}
```
Несмотря на то, что мы изменили только метод `Create()`, удобнее определить все методы для каждого сервиса в своём файле.
Теперь нужно указать все файлы `.proto` в нашем `CountryService.gRPC.csproj`-файле:
```xml
<!-- начало csproj -->
  <ItemGroup>
    <Protobuf Include="Protos\v1\country.proto" GrpcServices="Server" />
    <Protobuf Include="Protos\v1\country.browser.proto" GrpcServices="Server" />
    <Protobuf Include="Protos\v1\country.shared.proto" GrpcServices="Server" />
  </ItemGroup>
<!-- конец csproj -->
```
И то же самое проделать для проекта `CountryWiki.DAL.csproj`:
```xml
<!-- начало csproj -->
  <ItemGroup>
    <Protobuf Include="Protos\v1\country.proto" GrpcServices="Client" />
    <Protobuf Include="Protos\v1\country.shared.proto" GrpcServices="Client" />
  </ItemGroup>
<!-- конец csproj -->
```
Далее добавим реализацию `CountryGrpcServiceBrowser`:
```csharp
namespace CountryService.gRPC.Services;

public class CountryGrpcServiceBrowser : CountryServiceBrowserBase
{
    private readonly ICountryServices _countryService;

    public CountryGrpcServiceBrowser(ICountryServices countryService)
    {
        _countryService = countryService;
    }
    
//остальные методы такие же как в CountryGrpcService

    public override async Task Create(CountriesCreationRequest request, IServerStreamWriter<CountryCreationReply> responseStream, ServerCallContext context)
    {
        foreach (var countryToCreate in request.Countries)
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
> [!Note] Примечание
> Да, тут есть дублирование кода, но это мы сделали специально, расчитывая, что в дальнейшем класс будет развиваться независимо. Впрочем, можно и вынести общие методы в отдельный файл — подумайте самостоятельно, как это можно сделать.

Мы почти закончили! Теперь нужно зарегистрировать `CountryGrpcServiceBrowser` как сервис gRPC и включить поддержку [[cors|CORS]]. Включить поддержку CORS необходимо для работы gRPC-web в браузере. Здесь мы поступим просто, и разрешим все методы, все источники и все заголовки. Нужно помнить, что для корректной работы gRPC-web нужны такие дополнительные заголовки:
- `Grpc-Status`;
- `Grpc-Message`;
- `Grpc-Encoding`;
- `Grpc-Accept-Encoding`.

Вот как поменяется файл **Program.cs** проекта `CountryService.gRPC`:
```csharp
//... неизменившееся начало файла ...
// Добавляем CORS с именованной политикой
builder.Services.AddCors(o => o.AddPolicy("AllowAll", builder =>
{
    builder
        .AllowAnyOrigin()
        .AllowAnyMethod()
        .AllowAnyOrigin()
        .WithExposedHeaders("Grpc-Status", "Grpc-Message", "Grpc-Encoding", "Grpc-Accept-Encoding");
}));

var app = builder.Build();
// Вызываем UseCors с нашей политикой
app.UseCors("AllowAll");
app.UseGrpcWeb(new GrpcWebOptions { DefaultEnabled = true });
app.MapGrpcReflectionService();
app.MapGrpcService<CountryGrpcService>();
// Добавляем новый сервис gRPC
app.MapGrpcService<CountryGrpcServiceBrowser>();
//... неизменившийся конец файла
```