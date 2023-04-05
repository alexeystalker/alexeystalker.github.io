---
share: true
aliases:
 - gRPCurl
tags:
 - gRPC/gRPCUrl
 - gRPC/testing
---
# gRPCUrl
gRPCUrl — замечательный инструмент, позволяющий обращаться к [[endpoint|эндпоинтам]] gRPC, а также получать список доступных сервисов с их определениями. Для этого инструменту нужно узнать о определениях сервисов (`.proto`!). Для этого есть два пути:
- передать файлы `.proto` как аргументы;
- использовать рефлексию при помощи *gRPC Reflection*.

Далее мы будем рассматривать только второй способ.
Прежде всего, необходимо установить NuGet-пакет `Grpc.AspNetCore.Server.Reflection`. Далее в **Program.cs** регистрируем сервисы и применяем их перед нашими эндпоинтами:
```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddGrpc();
builder.Services.AddGrpcReflection(); //<---
builder.Services.AddSingleton<CountryManagementService>();

var app = builder.Build();

app.MapGrpcReflectionService(); //<---
app.MapGrpcService<CountryGrpcService>();

app.MapGet("/", () => "Communication with gRPC endpoints must be made through a gRPC client. To learn how to create a client, visit: https://go.microsoft.com/fwlink/?linkid=2086909");

app.Run();
```
gRPCurl был написан на Go, поэтому сперва нам надо поставить Go [отсюда](https://go.dev/doc/install), потом, убедившись, что Go SDK установлен командой
```bash
go version
```
выполним следующую команду (возможно, нужно выполнять в консоли Powershell)
```bash
go install github.com/fullstorydev/grpcurl/cmd/grpcurl@latest
```
Это установить последнюю версию инструмента и добавит его в PATH. Ну или можно скачать бинарник [отсюда](https://github.com/fullstorydev/grpcurl/releases), но тогда в PATH придётся добавлять его самостоятельно. Проверим, что всё хорошо, командой
```bash
grpcurl -help
```
Теперь запустим наше приложение CountryService, после чего вызовем команду
```bash
grpcurl localhost:%тут порт приложения% list
```
В результате получим
```
gRPCDemo.v1.CountryService
grpc.reflection.v1alpha.ServerReflection
```
В первой строчке будет наш сервис `gRPCDemo.v1.CountryService`, ведь именно `gRPCDemo.v1` мы указали в [[ch-5-create-and-compile-protobuf-files|нашем файле Protobuf]]. Теперь снова вызовем команду `list` с указанием сервиса:
```bash
grpcurl localhost:%тут порт приложения% list gRPCDemo.v1.CountryService
```
Получим список методов:
```
gRPCDemo.v1.CountryService.Create
gRPCDemo.v1.CountryService.Delete
gRPCDemo.v1.CountryService.Get
gRPCDemo.v1.CountryService.GetAll
gRPCDemo.v1.CountryService.Update
```
Теперь попробуем узнать определение команды `Create`. Для этого воспользуемся методом `describe`:
```bash
grpcurl localhost:%тут порт приложения% describe gRPCDemo.v1.CountryService.Create
```
получим:
```
gRPCDemo.v1.CountryService.Create is a method:
rpc Create ( stream .gRPCDemo.v1.CountryCreationRequest ) returns ( stream .gRPCDemo.v1.CountryCreationReply );
```
Да, если вызвать команду `describe` с указанием сервиса, получим больше информации, чем с командой `list`:
```
gRPCDemo.v1.CountryService is a service:
service CountryService {
  rpc Create ( stream .gRPCDemo.v1.CountryCreationRequest ) returns ( stream .gRPCDemo.v1.CountryCreationReply );
  rpc Delete ( stream .gRPCDemo.v1.CountryIdRequest ) returns ( .google.protobuf.Empty );
  rpc Get ( .gRPCDemo.v1.CountryIdRequest ) returns ( .gRPCDemo.v1.CountryReply );
  rpc GetAll ( .google.protobuf.Empty ) returns ( stream .gRPCDemo.v1.CountryReply );
  rpc Update ( .gRPCDemo.v1.CountryUpdateRequest ) returns ( .google.protobuf.Empty );
}
```
И наконец, посмотрим вывод команды `describe` для сообщения `CountryCreationRequest`:
```
gRPCDemo.v1.CountryCreationRequest is a message:
message CountryCreationRequest {
  string Name = 1;
  string Description = 2;
  bytes Flag = 3;
  .google.protobuf.Timestamp CreateDate = 5;
}
```
Теперь попробуем протестировать наши методы! Для формирования параметров будем использовать формат JSON согласно [документации](https://protobuf.dev/programming-guides/proto3/#json). Сперва вызовем метод `GetAll`, потому что он не требует параметров (сообщение [[ch-4-empty-messages|Empty]]):
```bash
grpcurl localhost:%тут порт приложения% gRPCDemo.v1.CountryService/GetAll
```
При тестировании указываем сервис и метод в виде `{serviceFullName}/{FunctionName}`.
Результат выполнения команды:
```
{
  "Id": 1,
  "Name": "Canada",
  "Description": "Canada has at least 32 000 lakes",
  "CreateDate": "2021-01-02T00:00:00Z"
}
{
  "Id": 2,
  "Name": "USA",
  "Description": "Yellowstone has 300 to 500 geysers",
  "CreateDate": "2021-01-02T00:00:00Z"
}
{
  "Id": 3,
  "Name": "Mexico",
  "Description": "Mexico is crossed by Sierra Madre Oriental and Sierra Madre Occidental mountains",
  "CreateDate": "2021-01-02T00:00:00Z"
}
```
Теперь попробуем выполнить [[ch-3-types-of-grpc-services|унарную]] операцию (формат для командной строки Windows, тут нам пришлось использовать двойные кавычки для аргумента, что повлекло за собой сдваивание двойных кавычек внутри)
```bash
grpcurl.exe -d "{\""Id\"":1}" localhost:%тут порт приложения% gRPCDemo.v1.CountryService/Get
```
Результат:
```
{
  "Id": 1,
  "Name": "Canada",
  "Description": "Canada has at least 32 000 lakes",
  "CreateDate": "2021-01-02T00:00:00Z"
}
```
В качестве последнего теста удалим две записи (это будет пример клиентского стриминга):
```
 grpcurl.exe -d "{\""Id\"":1}{\""Id\"":2}" localhost:7282 gRPCDemo.v1.CountryService/Delete
```
В ответ придёт пустое сообщение `{ }`.
Больше информации об использовании инструмента [здесь](https://github.com/fullstorydev/grpcurl#installation)