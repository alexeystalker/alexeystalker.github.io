---
share: true
tags:
 - gRPC/gRPC-web
---
# gRPC-web и клиенты .NET
Рассмотрим, как должен выглядеть клиент, не являющийся приложением ASP.NET Core или являющийся приложением ASP.NET Core 3.0 или ниже.
Такие клиенты используют пакет Grpc.Net.Client.Web. Этот пакет собран для .NET Standard 2.0, .NET Standard 2.1, .NET 5 и .NET 6.
```powershell
Install-Package Grpc.Net.Client.Web
```
Теперь создадим клиента
```csharp
public static IServiceCollection AddCountryServiceGrpcWebClient(this IServiceCollection services,
	ILoggerFactory loggerFactory, string countryServiceUri)
{
	var channel = GrpcChannel.ForAddress(new Uri(countryServiceUri), new GrpcChannelOptions
	{
		LoggerFactory = loggerFactory,
		CompressionProviders = new List<ICompressionProvider>
		{
			new BrotliCompressionProvider()
		},
		MaxReceiveMessageSize = 6291456,
		MaxSendMessageSize = 6291456,
		
		HttpHandler = new GrpcWebHandler(new HttpClientHandler())
	});
	// Просто создадим клиента
	// var client = new CountryServiceClient(channel);
	
	// Добавим перехватчик
	var client = new CountryServiceClient(channel
		.Intercept(new TracerInterceptor(loggerFactory.CreateLogger<TracerInterceptor>())));
	//Для простоты добавим синглтоном.
	services.AddSingleton(client);

	return services;
}
```
> [!Note] От меня
> В книге код написан так, как будто мы пишем отдельное консольное приложение. Я же, чтобы сократить количество добавляемого кода, создал ещё один метод расширения в библиотеке CountryWiki.DAL, о том, почему я решил сделать метод расширения, написано во врезке [[ch-9-create-and-configure-the-grpc-client-with-the-ihttpclientfactory-and-register-all-dependencies-in-the-program-cs-file|здесь]].

Основное интересующее нас действие в этом коде — создание `GrpcWebHandler`-а. Так как всё работает через HTTP, мы передаём `HttpClientHandler`. Вообще при создании `GrpcWebHandler` можно задать такие опции[^1]:
- `InnerHandler` — внутренний обработчик HTTP;
- `GrpcWebMode` — параметр, определяющий заголовок `Content-Type`. Возможные варианты: `GrpcWebMode.GrpcWebText` (по умолчанию, необходим для [[grpc-server-streaming-call|серверных потоков]]), `GrpcWebMode.GrpcWeb` можно использовать в случае, если у вас только [[grpc-unary-call|унарные вызовы]][^2]
- `HttpVersion` для указания версии HTTP. Принимает объект `Version`. Возможные варианты: `1.1` для HTTP/1, `2.0` для [[http-2|HTTP/2]]. По умолчанию проверяется, если клиент поддерживает HTTP/2, используется эта версия протокола, если нет — откатывается на HTTP/1. Например, `HttpClient` для `Xamarin.Android` основывается на `AndroidClientHandler`, который не поддерживает HTTP/2.

Вот как будет выглядеть инициализация с явным указанием опций:
```csharp
var channel = GrpcChannel.ForAddress(new Uri(countryServiceUri), new GrpcChannelOptions
{
	LoggerFactory = loggerFactory,
	CompressionProviders = new List<ICompressionProvider>
	{
		new BrotliCompressionProvider()
	},
	MaxReceiveMessageSize = 6291456,
	MaxSendMessageSize = 6291456,
	
	HttpHandler = new GrpcWebHandler
	{
		HttpVersion = new Version("1.1"),
		GrpcWebMode = GrpcWebMode.GrpcWebText,
		InnerHandler = new HttpClientHandler()
	}
});
```

[^1]: [Документация Microsoft](https://learn.microsoft.com/ru-ru/aspnet/core/grpc/grpcweb?view=aspnetcore-6.0#configure-grpc-web-with-the-net-grpc-client)
[^2]: Про разницу двух заголовков `Content-Type` сказано [[ch-10-history-and-specification-of-grpc-web|тут]]