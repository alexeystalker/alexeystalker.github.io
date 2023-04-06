---
share: true
tags:
 - gRPC/interceptors
---
# Перехватчики
Перехватчик - это своего рода промежуточное ПО, выполняемое для каждого gRPC-запроса, и только для gRPC-запроса. Промежуточное ПО ASP.NET Core выполняется *до* выполнения перехватчиков. Во время работы перехватчиков сообщение ещё не сериализовано, тогда как промежуточное ПО ASP.NET Core получает доступ к потоку байтов.
Перехватчик — это класс, унаследованный от абстрактного класса `Interceptor`, и может быть применён как глобально, так и точечно, для конкретного сервиса. Можно определить несколько перехватчиков, и они будут выполнены в том порядке, в котором были объявлены.
Класс `Interceptor` предоставляет четыре доступных для переопределения виртуальных метода:
- `UnaryServerHandler`
- `ClientStreamingServerHandler`
- `ServerStreamingServerHandler`
- `DuplexStreamingServerHandler`

Отметим, что перехватчики поддерживают [[dependency-injection-pattern|внедрение зависимостей]], и поэтому в них можно внедрить `ILogger`. По умолчанию перехватчики имеют время жизни `Per request`. [^1]

Создадим перехватчик, заменяющий возникающее исключение на `RpcException`:
```csharp
using CountryService.Web.Interceptors.Helpers;
using Grpc.Core;
using Grpc.Core.Interceptors;

namespace CountryService.Web.Interceptors;

public class ExceptionInterceptor : Interceptor
{
    private readonly ILogger<ExceptionInterceptor> _logger;
    private readonly Guid _correlationId;

    public ExceptionInterceptor(ILogger<ExceptionInterceptor> logger)
    {
        _logger = logger;
        _correlationId = Guid.NewGuid();
    }
    public override async Task<TResponse> UnaryServerHandler<TRequest, TResponse>(
        TRequest request,
        ServerCallContext context,
        UnaryServerMethod<TRequest, TResponse> continuation)
    {
        try
        {
            return await continuation(request, context);
        }
        catch (Exception e)
        {
            throw e.Handle(context, _logger, _correlationId);
        }
    }
    public override async Task<TResponse> ClientStreamingServerHandler<TRequest, TResponse>(
        IAsyncStreamReader<TRequest> requestStream, 
        ServerCallContext context,
        ClientStreamingServerMethod<TRequest, TResponse> continuation)
    {
        try
        {
            return await continuation(requestStream, context);
        }
        catch (Exception e)
        {
            throw e.Handle(context, _logger, _correlationId);
        }
    }
    public override async Task ServerStreamingServerHandler<TRequest, TResponse>(
        TRequest request, 
        IServerStreamWriter<TResponse> responseStream,
        ServerCallContext context,
        ServerStreamingServerMethod<TRequest, TResponse> continuation)
    {
        try
        {
            await continuation(request, responseStream, context);
        }
        catch (Exception e)
        {
            throw e.Handle(context, _logger, _correlationId);
        }
    }
    public override async Task DuplexStreamingServerHandler<TRequest, TResponse>(
        IAsyncStreamReader<TRequest> requestStream,
        IServerStreamWriter<TResponse> responseStream,
        ServerCallContext context,
        DuplexStreamingServerMethod<TRequest, TResponse> continuation)
    {
        try
        {
            await continuation(requestStream, responseStream, context);
        }
        catch (Exception e)
        {
            throw e.Handle(context, _logger, _correlationId);
        }
    }
}
```
Здесь метод `Handle` - это метод расширения для исключения, как раз и преобразующий возникающее исключение в соответствующее `RpcException`. Вот его код:
```csharp
using Grpc.Core;
using Microsoft.Data.SqlClient;

namespace CountryService.Web.Interceptors.Helpers;

public static class ExceptionHelpers
{
    public static RpcException Handle(
        this Exception exception,
        ServerCallContext context,
        ILogger logger,
        Guid correlationId) =>
        exception switch
        {
            TimeoutException timeoutException => HandleTimeoutException(timeoutException, context, logger, correlationId),
            SqlException sqlException => HandleSqlException(sqlException, context, logger, correlationId),
            RpcException rpcException => HandleRpcException(rpcException, context, logger, correlationId),
            _ => HandleDefault(exception, context, logger, correlationId)
        };
    private static RpcException HandleTimeoutException(TimeoutException exception, ServerCallContext context, ILogger logger, Guid correlationId)
    {
        logger.LogError(exception, "CorrelationId: {correlationId} - A timeout occurred", correlationId);
        var status = new Status(StatusCode.Internal, "An external resource did not answer within the time limit");
        return new RpcException(status, CreateTrailers(correlationId));
    }
    private static RpcException HandleSqlException(SqlException exception, ServerCallContext context, ILogger logger, Guid correlationId)
    {
        logger.LogError(exception, "CorrelationId: {correlationId} - A timeout occurred", correlationId);
        var status = exception.Number == -2
            ? new Status(StatusCode.DeadlineExceeded, "SQL timeout")
            : new Status(StatusCode.Internal, "SQL Error");
        return new RpcException(status, CreateTrailers(correlationId));
    }
    private static RpcException HandleRpcException(RpcException exception, ServerCallContext context, ILogger logger, Guid correlationId)
    {
         logger.LogError(exception, "CorrelationId: {correlationId} - An error occurred", correlationId);
        var trailers = exception.Trailers;
        // см. сноску 2
        var newTrailers = CreateTrailers(correlationId);
        foreach (var entry in trailers)
        {
            newTrailers.Add(entry);
        }
        return new RpcException(new Status(exception.Status.StatusCode, exception.Status.Detail), newTrailers);
    }
    private static RpcException HandleDefault(Exception exception, ServerCallContext context, ILogger logger, Guid correlationId)
    {
        logger.LogError(exception, "CorrelationId: {correlationId} - An error occurred", correlationId);
        return new RpcException(new Status(StatusCode.Internal, exception.Message), CreateTrailers(correlationId));
    }
    private static Metadata CreateTrailers(Guid correlationId)
    {
        var trailers = new Metadata();
        trailers.Add("CorrelationId", correlationId.ToString());
        return trailers;
    }
}
```
Здесь мы реализовали специальные методы для исключений с типами `TimeoutException`, `SqlException` и `RpcException` — на случай, если наше приложение будет подключаться к другому gRPC-сервису в качестве клиента[^2].
> [!Note]- От меня
> В книге класс назван `ExceptionHelpers`. Я обычно называю такие классы `ExceptionExtensions`, потому что он содержит методы расширения. Однако, сейчас будем следовать написанному в книге.

Осталось добавить наш перехватчик в общую коллекцию перехватчиков:
```csharp
builder.Services.AddGrpc(options =>
{
	// другие опции...
    options.Interceptors.Add<ExceptionInterceptor>(); //регистрируем наш перехватчик
});
```

[^1]: Время жизни можно изменить — см. [справку Microsoft](https://learn.microsoft.com/en-us/aspnet/core/grpc/interceptors?view=aspnetcore-6.0)
[^2]:	В книге, а также  [блоге автора](https://anthonygiretti.com/2022/08/28/asp-net-core-6-handling-grpc-exception-correctly-server-side/) приведён такой код:
	```csharp
	private static RpcException HandleRpcException<T>(RpcException exception, ILogger<T> logger, Guid correlationId)
	{
		logger.LogError(exception, $"CorrelationId: {correlationId} - An error occurred");
		var trailers = exception.Trailers;
		trailers.Add(CreateTrailers(correlationId)[0]);
		return new RpcException(new Status(exception.StatusCode, exception.Message), trailers);
	}
	```
	Но при попытке его использовать я получал исключение `System.InvalidOperationException: Object is read only` в строке `trailers.Add(CreateTrailers(correlationId)[0]);`. Не знаю, проверял ли автор свой код, но я выложу ту версию, которая сработала для меня.
