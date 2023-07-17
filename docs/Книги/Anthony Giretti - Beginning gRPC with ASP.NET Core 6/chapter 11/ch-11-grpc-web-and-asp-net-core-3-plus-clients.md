---
share: true
tags:
 - gRPC/gRPC-web
---
# gRPC-web и клиенты ASP.NET Core 3+
Теперь перейдём к клиентам с ASP.NET Core версии не ниже 3. Для них настройка выполняется немного по-другому, с использованием фабрики.
Сперва установим NuGet-пакет `Grpc.Net.Client.Web`, затем добавим следующее:
```csharp
public static IServiceCollection AddCountryServiceGrpcWebAspNetCoreClient(
	this IServiceCollection services,
	ILoggerFactory loggerFactory,
	string countryServiceUri)
{
	services.AddGrpcClient<CountryServiceClient>(o =>
		{
			o.Address = new Uri(countryServiceUri);
		})
		//Добавляем GrpcWebHandler здесь
		.ConfigurePrimaryHttpMessageHandler(() => new GrpcWebHandler(new HttpClientHandler()))
		.AddInterceptor(() => new TracerInterceptor(loggerFactory.CreateLogger<TracerInterceptor>()))
		.ConfigureChannel(o =>
		{
			o.CompressionProviders = new List<ICompressionProvider>
			{
				new BrotliCompressionProvider()
			};
			o.MaxReceiveMessageSize = 6291456;
			o.MaxSendMessageSize = 6291456;
		});

	return services;
}
```
> [!Note] От меня
> Как уже было [[ch-11-grpc-web-and-all-net-clients|здесь]] и [[ch-9-create-and-configure-the-grpc-client-with-the-ihttpclientfactory-and-register-all-dependencies-in-the-program-cs-file|здесь]], я оформляю код в виде метода расширения внутри проекта `CountryWiki.DAL`.

Здесь мы перенесли назначение обработчика `GrpcWebHandler` в метод расширения `ConfigurePrimaryHttpMessageHandler`, а затем продолжили цепочку, как раньше.