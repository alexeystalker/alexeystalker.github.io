---
share: true
tags:
 - gRPC/validation
---
# Получаем ошибки валидации сообщений на сервере

В [[ch-5-perform-message-validation|Главе 5]] мы рассмотрели, как можно преодолеть нехватку встроенной валидации сообщений в ASP.NET Core gRPC. Автор предлагает решение, которое отправляет ошибки валидации в структурированной форме, которая содержит:
- имя свойства, не прошедшего валидацию;
- значение свойства, не прошедшее валидацию;
- сообщение об ошибке.

Для того, чтобы легче было получить и интерпертировать такие сообщения, автор подготовил клиентский NuGet-пакет `Calzolari.Grpc.Net.Client.Validation`. Этот пакет предоставляет метод расширения `GetValidationErrors()`. Вот как предлагается его использовать:
```csharp
async Task CreateWithValidationError(CountryServiceClient client, ILogger logger)
{
    using var bidirectionalStreamingCall = client.Create();
    var countriesToCreate = new List<CountryCreationRequest>
    {
        new CountryCreationRequest
        {
            Name = "Japan",
            Description = "", // Нарушает правило "минимум 5 символов
            CreateDate =  Timestamp.FromDateTime(DateTime.SpecifyKind(DateTime.UtcNow, DateTimeKind.Utc))
        }
    };
    try
    {
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
    }
    catch (RpcException ex) when (ex.StatusCode == StatusCode.InvalidArgument)
    {
        var errors = ex.GetValidationErrors();
        logger.LogWarning($"validation error message: {ex.Message}{Environment.NewLine}Errors:{Environment.NewLine}{JsonSerializer.Serialize(errors, new JsonSerializerOptions { WriteIndented = true })}");
    }
    catch (Exception e)
    {
        logger.LogWarning(e, e.Message);
    }
}
```
Если мы запустим этот запрос, то увидим в логах такую запись уровня `Warn`:
```log
warn: CreateWithValidationError[0]
      validation error message: Status(StatusCode="InvalidArgument", Detail="Property Description failed validation. Error code was: MinimumLengthValidator Error was: Description is mandatory and should be longer than 4 characters")
      Errors:
      [
        {
          "PropertyName": "Description",
          "ErrorMessage": "Description is mandatory and should be longer than 4 characters",
          "AttemptedValue": ""
        }
      ]
```