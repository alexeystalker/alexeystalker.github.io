---
share: true
tags:
 - gRPC/duplex-streaming
---
# Двунаправленный поток
Следующий пример — вызов метода `Create()`, являющегося двунаправленной потоковой функцией. Это комбинация клиентского и серверного потоков. Двунаправленный поток также требует вызова `CompleteAsync()` для указания того, что передача завершена. Заголовки могут быть прочитаны до завершения чтения серверного потока, но трейлеры — строго после. Обратите внимание, что для простоты мы сначала отправляем все сообщения на сервер, но имейте в виду, что можно начать вычитывать сообщения с сервера до окончания передачи.
```csharp
async Task Create(CountryServiceClient client, ILogger logger)
{
    using var bidirectionalStreamingCall = client.Create();
    var countriesToCreate = new List<CountryCreationRequest>
    {
        new CountryCreationRequest
        {
            Name = "France",
            Description = "Western european country",
            CreateDate = Timestamp.FromDateTime(DateTime.SpecifyKind(DateTime.UtcNow, DateTimeKind.Utc))
        },
        new CountryCreationRequest
        {
            Name = "Poland",
            Description = "Eastern european country",
            CreateDate = Timestamp.FromDateTime(DateTime.SpecifyKind(DateTime.UtcNow, DateTimeKind.Utc))
        }
    };
    // Записываем
    foreach (var request in countriesToCreate)
    {
        await bidirectionalStreamingCall.RequestStream.WriteAsync(request);
        logger.LogInformation($"Country {request.Name} set for creation");
    }
    // Сообщаем серверу о завершении передачи
    await bidirectionalStreamingCall.RequestStream.CompleteAsync();
    // Читаем поток с сервера
    await foreach (var createdCountry in bidirectionalStreamingCall.ResponseStream.ReadAllAsync())
    {
        logger.LogInformation($"{createdCountry.Name} has been created with Id {createdCountry.Id}");
    }
    // Читаем заголовки и трейлеры, сериализуем в JSON и пишем в лог
    var bidirectionalStreamingCallHeaders = await bidirectionalStreamingCall.ResponseHeadersAsync;
    logger.LogInformation($"Headers:{Environment.NewLine}{JsonSerializer.Serialize(bidirectionalStreamingCallHeaders, new JsonSerializerOptions { WriteIndented = true })}");

    var bidirectionalStreamingCallTrailers = bidirectionalStreamingCall.GetTrailers();
    logger.LogInformation($"Trailers:{Environment.NewLine}{JsonSerializer.Serialize(bidirectionalStreamingCallTrailers, new JsonSerializerOptions { WriteIndented = true })}");
}
```