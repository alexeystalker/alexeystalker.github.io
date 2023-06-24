---
share: true
tags:
 - gRPC/client-streaming
---
# Клиентский поток
Далее рассмотрим вызов `Delete()` с потоком сообщений с клиента. По завершению передачи необходимо вызвать `CompleteAsync()`. Далее, чтобы прочитать заголовки И [[ch-3-trailers|трейлеры]], нужно выполнить `await ResponseAsync`, при этом заголовки можно прочитать до вызова этого метода. Если заголовки и трейлеры не нужны, можно использовать `await clientStreamingCall`.
```csharp
async Task Delete(CountryServiceClient client, ILogger logger)
{
    using var clientStreamingCall = client.Delete();
    var countriesToDelete = new List<CountryIdRequest>
    {
        new CountryIdRequest {Id = 1},
        new CountryIdRequest {Id = 2}
    };
    // Записываем
    foreach (var request in countriesToDelete)
    {
        await clientStreamingCall.RequestStream.WriteAsync(request);
        logger.LogInformation($"Country with id {request.Id} set for deletion");
    }
    // Сообщаем серверу о завершении передачи
    await clientStreamingCall.RequestStream.CompleteAsync();
    // Завершаем запрос получением ответа
    var emptyResponse = await clientStreamingCall.ResponseAsync;
    // Читаем заголовки и трейлеры, сериализуем в JSON и пишем в лог
    var clientStreamingCallHeaders = await clientStreamingCall.ResponseHeadersAsync;
    logger.LogInformation($"Headers:{Environment.NewLine}{JsonSerializer.Serialize(clientStreamingCallHeaders, new JsonSerializerOptions { WriteIndented = true })}");

    var clientStreamingCallTrailers = clientStreamingCall.GetTrailers();
    logger.LogInformation($"Trailers:{Environment.NewLine}{JsonSerializer.Serialize(clientStreamingCallTrailers, new JsonSerializerOptions { WriteIndented = true })}");

    //await clientStreamingCall; //Так тоже можно завершить, но тогда заголовки и трейлеры не прочитать.
}

```