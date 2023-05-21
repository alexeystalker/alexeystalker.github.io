---
share: true
tags:
 - gRPC
---
# Опция IgnoreUnknownServices
Среди настроек gRPC есть настройка `IgnoreUnknownServices`. Она отвечает за обработку gRPC-запросов к неизвестным сервисам или методам. Без неё (или если `IgnoreUnknownServices = false`) фреймворк gRPC будет возвращать статус `Unimplemented` сразу, не проходя весь конвейер целиком. Если же её включить - запрос пройдёт дальше по конвейеру ASP.NET.
> [!Note] От меня
> Чтобы попробовать настройку в действии, сделаем следующее. Скопируем куда-нибудь наш файл **country.proto** и в копии добавим ещё один метод. Например
> ```protobuf
> service CountryService {
>	rpc GetAll(google.protobuf.Empty) returns (stream CountryReply) {}
>	rpc GetFooBar(google.protobuf.Empty) returns (google.protobuf.Empty) {} //<-- новый метод
>	rpc Get(CountryIdRequest) returns (CountryReply) {}
>	rpc Delete(stream CountryIdRequest) returns (google.protobuf.Empty) {}
>	rpc Update(CountryUpdateRequest) returns (google.protobuf.Empty) {}
>	rpc Create(stream CountryCreationRequest) returns (stream CountryCreationReply) {}
> }
> ```
> Теперь запустим `grpcui`, отключив чтение рефлексии и используя только модифицированный файл:
> ```bash
> grpcui -use-reflection=false -proto=country.proto localhost:%порт_приложения%
> ```
> В списке доступных методов появится наш метод `GetFooBar`, хотя в приложении он заведомо не реализован, даже в сгенерированном коде.

Вот что будет, если мы обратимся к сервису с `IgnoreUnknownServices = false`:
![[Pasted image 20230405171527.png]]
Теперь, чтобы убедиться в том, что обработка запроса проходит дальше по [[ch-3-combining-middleware-in-a-pipeline|конвейеру промежуточного ПО]], добавим промежуточное ПО, выставляющее gRPC-статус `NotFound`:
```csharp
//Выше предыдущий код Program.cs

app.MapGet("/", () => "Communication with gRPC endpoints must be made through a gRPC client. To learn how to create a client, visit: https://go.microsoft.com/fwlink/?linkid=2086909");

app.Use(async(context, next) =>
{
    if (context.Request.ContentType == "application/grpc")
    {
        context.Response.OnStarting(() =>
        {
            if (context.Response.StatusCode == 404)
            {
                context.Response.ContentType = "application/grpc";
                context.Response.Headers.Add("grpc-status", ((int)StatusCode.NotFound).ToString());
                context.Response.StatusCode = 200; //Помним о том, что HTTP-код должен быть 200 OK
            }
            return Task.CompletedTask;
        });
    }
    await next(context);
});

app.Run();
```
Запускаем, видим в gRPCUi следующее:
![[Pasted image 20230405171341.png]]
