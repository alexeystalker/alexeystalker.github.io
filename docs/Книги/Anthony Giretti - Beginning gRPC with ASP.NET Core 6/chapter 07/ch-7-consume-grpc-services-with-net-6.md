---
share: true
tags:
 - gRPC/channel
 - NET/NET6
---
# Используем сервисы gRPC в NET 6
В этом разделе напишем простой gRPC клиент. Рассмотрим самые основы.
Создание клиента выглядит следующим образом (используя *top-level statements* в файле **Program.cs**):
```csharp
using Grpc.Net.Client;
using static CountryService.Web.gRPC.v1.CountryService;

var channel = GrpcChannel.ForAddress("https://localhost:7282");
var countryClient = new CountryServiceClient(channel);
```
Это простейший способ. Также можно добавить в вызов `ForAddress` несколько опций. Например — логирование. Сперва добавим в приложение NuGet-пакет `Microsoft.Extensions.Logging.Console`, затем создадим `LoggerFactory` и добавим в опции. Получится вот что:
```csharp
using Grpc.Net.Client;
using Microsoft.Extensions.Logging;
using static CountryService.Web.gRPC.v1.CountryService;

var loggerFactory = LoggerFactory.Create(logging =>
{
    logging.AddConsole();
    logging.SetMinimumLevel(LogLevel.Trace);
});
var channel = GrpcChannel.ForAddress("https://localhost:7282", new GrpcChannelOptions
{
	LoggerFactory = loggerFactory
});
var countryClient = new CountryServiceClient(channel);
```
Теперь перейдём к использованию класса `CountryServiceClient`. Рассмотрим, как вызвать и затем использовать полученные данные следующих эндпоинтов:
- `GetAll` — [[grpc-server-streaming-call|серверный поток]];
- `Get` — [[grpc-unary-call|унарная функция]];
- `Create` — [[grpc-duplex-streaming-call|двунаправленный поток]];
- `Delete` — [[grpc-client-streaming-call|клиентский поток]].

Общий принцип работы с данными тот же, что и в серверной части.
- Потоковый ответ должен быть проитерирован при помощи метода `ReadAllAsync()`;
- Потоковый запрос должен быть проитерирован при помощи метода `WriteAsync()`.

Также у потоковых методов есть несколько особенностей:
- вызов функции gRPC является объектом, реализующим интерфейс `IDisposable`;
- вызов потоковой функции является асинхронным (awaitable), но не имеет суффикса *async*. Чтение и запись потокового ответа является асинхронным, равно как и завершение вызова;
- после завершения отправки сообщений потокового запроса необходимо сообщить серверу о завершении передачи с помощью метода `CompleteAsync()`;
- если вы ожидаете непотоковый ответ после завершения вызова метода с клиентским стримингом, необходимо использовать `await` на объекте вызова для получения финального ответного сообщения. Если никакого непотокового сообщения не предусмотрено, всё равно необходимо использовать `await` без получения данных. Наконец, использование `await`  на объекте вызова завершает вызов (передаёт сигнал серверу о том, что вызов завершён).

> [!Note] От меня
> В книге код каждого запроса используется отдельно (то есть для каждого запроса демострируется файл **Program.cs** целиком), я решил оформить в виде отдельных методов. Каждый метод получает клиента и логгер как параметры при вызове, вот так:
> ```csharp
> await GetAll(countryClient, loggerFactory.CreateLogger(nameof(GetAll)));
> ```

#### [[ch-7-server-streaming-call|Серверный поток]]
#### [[ch-7-client-streaming-call|Клиентский поток]]
#### [[ch-7-duplex-streaming-call|Двунаправленный поток]]
#### [[ch-7-unary-call|Унарный вызов]]
#### [[ch-7-deadlines|Дедлайны]]
#### [[ch-7-interceptors|Перехватчики]]

---
И последнее, о чём нужно упомянуть в этом разделе. Канал gRPC реализует `IDisposable`, поэтому метод `Dispose` должен вызываться, когда канал больше не нужен, при этом закрываются все активные вызовы. Также нужно вызвать метод `ShutdownAsync()` для того, чтобы разрегистрировать (unregister) канал.
Вот как это выглядит:
```csharp
var channel = GrpcChannel.ForAddress("https://localhost:7282");
var countryClient = new CountryServiceClient(channel);

// ... вызовы методов клиента ...

channel.Dispose();
await channel.ShutdownAsync();
```
