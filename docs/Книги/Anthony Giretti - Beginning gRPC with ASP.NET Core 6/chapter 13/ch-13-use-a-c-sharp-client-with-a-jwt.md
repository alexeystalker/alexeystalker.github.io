---
share: true
tags:
 - security
 - gRPC/client
---
# Используем клиент на C# с JWT
Теперь давайте посмотрим, как нам передавать JWT в запросе от клиента на C#.
Прежде всего предлагается реализовать обработку ошибок [[authentication|аутентификации]], то есть [[ch-3-grpc-status|gRPC-статусов]] `UNAUTHENTICATED` и `PERMISSIONDENIED` — это будет полезно для отладки проблем с аутентификацией на клиентской стороне. Следующий пример показывает, как использовать токен и передавать его в в запрос, добавляя заголовок `authorization` со значением `bearer {TOKEN}`, а также демонстрирует блок `try/catch` с использованием нескольких `catch when`.
```csharp
public async IAsyncEnumerable<CountryModel> GetAllAsync(string token)
{
	var list = new List<CountryModel>();
	try
	{
		var metadata = new Metadata();
		metadata.Add("authorization", $"bearer {token}");
		using var serverStreamingCall = _countryServiceClient.GetAll(new Empty(), metadata);
		while (await serverStreamingCall.ResponseStream.MoveNext(CancellationToken.None))
		{
			list.Add(serverStreamingCall.ResponseStream.Current.ToDomain());
		}
	}
	catch (RpcException ex) when (ex.StatusCode == StatusCode.Unauthenticated)
	{
		_logger.LogError(ex, "Token was missing or invalid");
	}
	catch (RpcException ex) when (ex.StatusCode == StatusCode.PermissionDenied)
	{
		_logger.LogError(ex, "Token was valid butt missed particular role");
	}
	catch (RpcException ex)
	{
		_logger.LogError(ex, "an error occured");
	}

	foreach (var country in list)
	{
		yield return country;
	}
}
```
