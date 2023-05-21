---
share: true
tags:
 - gRPC/server-streaming
---
# Серверный поток
Вот пример вызова `GetAll()` с асинхронным чтением серверного потока, доступом к заголовкам и трейлерам. Заголовки читаются при помощи свойства `ResponseHeadersAsync` (асинхронного), а трейлеры — при помощи синхронного метода `GetTrailers()`. Трейлеры необходимо читать после завершения чтения потока, заголовки можно прочесть до завершения чтения.
```csharp
async Task GetAll(CountryServiceClient client, ILogger logger)
{
    using var serverStreamingCall = client.GetAll(new Empty());
    await foreach (var response in serverStreamingCall.ResponseStream.ReadAllAsync())
    {
        logger.LogInformation($"{response.Name}: {response.Description}");
    }
    //Читаем заголовки и трейлеры, сериализуем в JSON и пишем в лог
    var serverStreamingCallHeaders = await serverStreamingCall.ResponseHeadersAsync;
    logger.LogInformation($"Headers:{Environment.NewLine}{JsonSerializer.Serialize(serverStreamingCallHeaders, new JsonSerializerOptions { WriteIndented = true })}");

    var serverStreamingCallTrailers = serverStreamingCall.GetTrailers();
    logger.LogInformation($"Trailers:{Environment.NewLine}{JsonSerializer.Serialize(serverStreamingCallTrailers, new JsonSerializerOptions { WriteIndented = true })}");
}
```