---
share: true
tags:
 - gRPC/compression
---
# Используем преимущество сжатия
Ранее мы рассмотрели, как [[ch-5-add-compression-provider|сконфигурировать сжатие на серверной стороне]]. Сжатие сконфигурировано, но не работает до тех пор, пока клиент не пришлёт заголовок с допустимыми методами сжатия, `HeaderGrpcAcceptEncoding`.
Чтобы проверить содержимое этого заголовка на серверной стороне, нужно обратиться к HTTP-контексту внутри обработчика вызова:
```csharp
        var headers = context.GetHttpContext().Request.Headers;
        headers.GrpcAcceptEncoding; //Одним из значений должно быть br для алгоритма Brotli
```
Чтобы использовать сжатие, добавим провайдер в настройках канала:
```csharp
var channel = GrpcChannel.ForAddress(
    "https://localhost:7282",
    new GrpcChannelOptions
    {
        CompressionProviders = new List<ICompressionProvider>
        {
            new BrotliCompressionProvider()
        }
    });
```
Если посмотреть логи, можно увидеть, что сообщения разжимаются алгоритмом `br`.
Также обратите внимание, что сжатию подвергаются только ответные сообщения, а не сообщеня запросов.