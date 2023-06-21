---
share: true
tags:
 - gRPC/interceptors
---
# Перехватчики
В клиенте gRPC сервиса можно использовать перехватчики — также как и [[ch-5-interceptors|на серверной стороне]]. Клиентские перехватчики имеют, в отличие от серверных, другие методы, которые нужно реализовывать, а именно:
- `AsyncClientStremingCall` — перехватчик для функций с [[ch-7-client-streaming-call|клиентским потоком]];
- `AsyncDuplexStreamingCall` — перехватчик для функций с [[ch-7-duplex-streaming-call|двунаправленным потоком]];
- `AsyncServerStreamingCall` — перехватчик для функций с [[ch-7-server-streaming-call|серверным потоком]];
- `AsyncUnaryCall` — перехватчик для [[ch-7-unary-call|унарных функций]].

Как и в случае с серверными перехватчиками, клиентский должен быть унаследован от класса `Interceptor`. Вот пример такого перехвачика, выполняющего кастомное логирование.
```csharp
using Grpc.Core;
using Grpc.Core.Interceptors;
using Microsoft.Extensions.Logging;

namespace CountryService.Client;

public class TracerInterceptor : Interceptor
{
    private readonly ILogger<TracerInterceptor> _logger;

    public TracerInterceptor(ILogger<TracerInterceptor> logger)
    {
        _logger = logger;
    }

    public override AsyncClientStreamingCall<TRequest, TResponse> AsyncClientStreamingCall<TRequest, TResponse>(
        ClientInterceptorContext<TRequest, TResponse> context,
        AsyncClientStreamingCallContinuation<TRequest, TResponse> continuation)
    {
        _logger.LogDebug($"Executing {context.Method.Name} {context.Method.Type} method on server {context.Host}");
        return continuation(context);
    }

    public override AsyncDuplexStreamingCall<TRequest, TResponse> AsyncDuplexStreamingCall<TRequest, TResponse>(
        ClientInterceptorContext<TRequest, TResponse> context,
        AsyncDuplexStreamingCallContinuation<TRequest, TResponse> continuation)
    {
        _logger.LogDebug($"Executing {context.Method.Name} {context.Method.Type} method on server {context.Host}");
        return continuation(context);
    }

    public override AsyncServerStreamingCall<TResponse> AsyncServerStreamingCall<TRequest, TResponse>(
        TRequest request,
        ClientInterceptorContext<TRequest, TResponse> context, 
        AsyncServerStreamingCallContinuation<TRequest, TResponse> continuation)
    {
        _logger.LogDebug($"Executing {context.Method.Name} {context.Method.Type} method on server {context.Host}");
        return continuation(request, context);
    }

    public override AsyncUnaryCall<TResponse> AsyncUnaryCall<TRequest, TResponse>(
        TRequest request, 
        ClientInterceptorContext<TRequest, TResponse> context,
        AsyncUnaryCallContinuation<TRequest, TResponse> continuation)
    {
        _logger.LogDebug($"Executing {context.Method.Name} {context.Method.Type} method on server {context.Host}");
        return continuation(request, context);
    }
}
```
Теперь уберём логгер из настроек канала, и добавим объект перехватчика:
```csharp
//...
var loggerFactory = LoggerFactory.Create(logging =>
{
    logging.AddSimpleConsole();
    logging.SetMinimumLevel(LogLevel.Debug);
});
var channel = GrpcChannel.ForAddress("https://localhost:7282");
var countryClient = new CountryServiceClient(
	channel.Intercept(new TracerInterceptor(loggerFactory.CreateLogger<TracerInterceptor>())));
//...
```
