---
share: true
tags:
 - gRPC/channel
 - HTTP-2
---
# Держим соединения HTTP/2 открытыми
Соединения HTTP/2 можно держать открытыми при помощи регулярных пингов. Так как открытие нового соединения требует некоторого времени, может стать хорошей идеей оставить текущее соединение открытым для будущих вызовов. В примере мы конфигурируем объект класса `SocketHttpHandler` отправлять пинги каждые 15 секунд, затем передаём его в свойство `HttpHandler` объекта настроек канала.
```csharp
var handler = new SocketsHttpHandler
{
    KeepAlivePingDelay = TimeSpan.FromSeconds(15)
};
var channel = GrpcChannel.ForAddress(
    "https://localhost:7282",
    new GrpcChannelOptions
    {
        LoggerFactory = loggerFactory,
        CompressionProviders = new List<ICompressionProvider>
        {
            new BrotliCompressionProvider()
        },
        MaxReceiveMessageSize = 6291456, // 6 Mb
        MaxSendMessageSize = 6291456, // 6 Mb
        HttpHandler = handler
    });
```
> [!attention] Внимание!
> Для начала отправки пингов *требуется* выполнить хотя бы один запрос; если просто создать объект канала или клиента, и не выполнить запроса на сервер, пинги не начнут отправляться.

Также есть возможность ограничить время простоя соединения; в следующем примере мы ограничим время простоя 5 минутами, после чего пинги перестанут посылаться. Также можно задать бесконечное время простоя.
```csharp
var handler = new SocketsHttpHandler
{
    KeepAlivePingDelay = TimeSpan.FromSeconds(15),
    PooledConnectionIdleTimeout = TimeSpan.FromMinutes(5)
        //Используйте Timeout.InfiniteTimeSpan для бесконечного таймаута,
};
```
Наконец, рекомендуется задавать таймаут ответа на пинги. В примере зададим таймаут ожидания ответа на пинг в 5 секунд
```csharp
var handler = new SocketsHttpHandler
{
    KeepAlivePingDelay = TimeSpan.FromSeconds(15),
    PooledConnectionIdleTimeout = TimeSpan.FromMinutes(5),
        //Используйте Timeout.InfiniteTimeSpan для бесконечного таймаута,
    KeepAlivePingTimeout = TimeSpan.FromSeconds(5)
};
```
> [! Note] От меня
> Чтобы увидеть пинги в логах сервера (а их можно увидеть только там), нужно задать уровень логирования `Trace` для сообщений Kestrel так:
> ```csharp
> builder.Logging.AddFilter("Microsoft.AspNetCore", LogLevel.Trace);
> ```
> Подробнее о настройке уровней логирования на основе фильтров написано [[ch-17-changing-log-verbosity-with-filtering|здесь]].