---
share: true
tags:
 - gRPC/unary
---
# Унарный метод
Следующий пример — вызов метода `Get()`. Эту функцию можно вызывать как синхронно, так и асинхронно — такова особенность унарных методов. Ответ приходет сразу после вызова операции. Заголовки можно прочитать до обращения к ` ResponseAsync`, трейлеры — строго после.
```csharp
async Task Get(CountryServiceClient client, ILogger logger)
{
    // создаём объект вызова
    var countryCall = client.GetAsync(new CountryIdRequest {Id = 1});
    // читаем ответ
    var country = await countryCall.ResponseAsync;
    logger.LogInformation($"{country.Id}: {country.Name}");
    // Читаем заголовки и трейлеры, сериализуем в JSON и пишем в лог
    var countryCallHeaders = await countryCall.ResponseHeadersAsync;
    logger.LogInformation($"Headers:{Environment.NewLine}{JsonSerializer.Serialize(countryCallHeaders, new JsonSerializerOptions { WriteIndented = true })}");

    var countryCallTrailers = countryCall.GetTrailers();
    logger.LogInformation($"Trailers:{Environment.NewLine}{JsonSerializer.Serialize(countryCallTrailers, new JsonSerializerOptions { WriteIndented = true })}");

    // альтернативный вариант:
    // var country = await client.GetAsync(new CountryRequest { Id = 1 });
    // Но при этом заголовки и трейлеры будут недоступны
}
```