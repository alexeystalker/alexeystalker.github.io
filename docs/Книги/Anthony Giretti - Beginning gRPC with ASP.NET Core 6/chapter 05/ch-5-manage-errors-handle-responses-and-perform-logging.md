---
share: true
tags:
 - gRPC
---
# Обрабатываем ошибки, управляем ответами и логированием

Рассмотрим, как обрабатывать возникающие в процессе обработки gRPC-запросов исключения.
Пусть метод `GetAll()` выбрасывает исключение, как на примере:
```csharp
public override async Task GetAll(Empty request, IServerStreamWriter<CountryReply> responseStream, ServerCallContext context)
{
	////---Выкинем исключение---////
	throw new Exception("Something got really wrong here");

	//Стримим все найденные страны клиенту
	var replies = await _countryManagementService.GetAllAsync();
	foreach (var countryReply in replies)
	{
		await responseStream.WriteAsync(countryReply);
	}
}
```
При попытке вызвать `GetAll` из [[ch-5-using-grpcui|gRPCUi]] получим ошибку сервера со статусом `Unknown`.
![[Pasted image 20230401195109.png]]
Как видим, текста ошибки мы на клиенте не получили (в логах сервера сообщение при этом отразилось). Мы никак не можем связать ошибку на клиентской стороне с тем, что случилось на серверной. Как это исправить?
Прежде всего, добавим опцию `EnableDetailedErrors` в инициализацию gRPC:
```csharp
builder.Services.AddGrpc(options =>
{
    options.EnableDetailedErrors = true; //<---
    // другие опции...
    // ...
});
```
Теперь сообщение исключения будет передано на клиент.

Далее, было бы желательно, чтобы статус был более информативным, нежели `UNKNOWN`. Для этого обернём код метода в `try/catch`, и в `catch`-блоке выбросим исключение `RpcException`, которое сможет вернуть осмысленный код статуса. Также добавим в трейлеры ответа уникальную строку `correlationId`: по этому уникальному ключу можно связать ошибку, возникшую на клиенте, с конкретной ошибкой на сервере. Заодно добавим логгер[^1], который залогирует исходную ошибку и `correlationId`.
```csharp
public class CountryGrpcService : CountryServiceBase
{
    private readonly CountryManagementService _countryManagementService;
    private readonly ILogger<CountryGrpcService> _logger;

    public CountryGrpcService(
	    CountryManagementService countryManagementService,
	    ILogger<CountryGrpcService> logger)
    {
        _countryManagementService = countryManagementService;
        _logger = logger;
    }

    public override async Task GetAll(Empty request, IServerStreamWriter<CountryReply> responseStream, ServerCallContext context)
    {
        try
        {
            ////---Выкинем исключение---////
            throw new Exception("Something got really wrong here");

            //Стримим все найденные страны клиенту
            var replies = await _countryManagementService.GetAllAsync();
            foreach (var countryReply in replies)
            {
                await responseStream.WriteAsync(countryReply);
            }
        }
        catch (Exception e)
        {
            var correlationId = Guid.NewGuid();
            _logger.LogError(e, "CorrelationId: {0}", correlationId);

            var trailers = new Metadata();
            trailers.Add("CorrelationId", correlationId.ToString()); //Добавим correlationId в трейлеры
            throw new RpcException(
                new Status(
	                StatusCode.Internal,
					$"Error message sent to the client with CorrelationId: {correlationId}"),
                trailers,
                "Error message that will appear in server log"); //на самом деле в логе нет
        }
    }
    //... другие методы ...
}
```
> [!attention] От меня
> Обратите внимание, что у RpcException нет конструктора с `InnerException`, А сообщение, передаваемое клиенту, задаётся в конструкторе объекта `Status`. 

Вот, что получит клиент:
![[Pasted image 20230401203248.png]]

Хорошо, но можно сделать ещё лучше при помощи *перехватчиков (interceptors)*
#### [[ch-5-interceptors|Перехватчики]]
#### [[ch-5-ignore-unknown-services|Опция IgnoreUnknownServices]]

---



[^1]: Про логгеры см. [[ch-17-adding-log-messages-to-app|здесь]]
