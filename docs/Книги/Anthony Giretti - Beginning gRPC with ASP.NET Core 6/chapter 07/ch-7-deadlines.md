---
share: true
tags:
 - gRPC/deadline
---
# Дедлайны
Ранее мы [[ch-3-deadline-and-cancellation|уже упоминали дедлайны]] — таймауты, задаваемые на стороне клиента. Рассмотрим на примере [[ch-7-unary-call|унарного вызова]], как задать дедлайн и обработать исключение, возникающее при достижении дедлайна.
```csharp
async Task Get(CountryServiceClient client, ILogger logger)
{
    var countryRequest = new CountryIdRequest {Id = 1};
    try
    {
        var countryCall = client.GetAsync(countryRequest);
        var country = await countryCall.ResponseAsync;
        logger.LogInformation($"{country.Id}: {country.Name}");
        var countryCallHeaders = await countryCall.ResponseHeadersAsync;
        logger.LogInformation(
            $"Headers:{Environment.NewLine}{JsonSerializer.Serialize(countryCallHeaders, new JsonSerializerOptions {WriteIndented = true})}");

        var countryCallTrailers = countryCall.GetTrailers();
        logger.LogInformation(
            $"Trailers:{Environment.NewLine}{JsonSerializer.Serialize(countryCallTrailers, new JsonSerializerOptions {WriteIndented = true})}");
    }
    catch (RpcException ex) when (ex.StatusCode == StatusCode.DeadlineExceeded)
    {
        var trailers = ex.Trailers;
        var correlationId = trailers.GetValue("correlationId");
        logger.LogWarning($"Get country with Id: {countryRequest.Id} has timed out, correlationId: {correlationId}");
    }
    catch (RpcException ex)
    {
        var trailers = ex.Trailers;
        var correlationId = trailers.GetValue("correlationId");
        logger.LogWarning($"An error occurred while getting the country with Id: {countryRequest.Id} has timed out, correlationId: {correlationId}");
    }
}
```