---
share: true
tags:
 - gRPC/channel
---
# Задаём ограничение размера сообщения
Можно ограничивать размер сообщения на стороне клиента также, как и [[ch-5-write-configure-and-expose-grpc-services|на стороне сервера]]. Настройка выполняется с помощью класса `GrpcChannelOptions`. 
```csharp
var channel = GrpcChannel.ForAddress(
    "https://localhost:7282",
    new GrpcChannelOptions
    {
        LoggerFactory = loggerFactory,
        CompressionProviders = new List<ICompressionProvider>
        {
            new BrotliCompressionProvider()
        },
        MaxReceiveMessageSize = 6291456, // 6 Mb
        MaxSendMessageSize = 6291456 // 6 Mb
    });
```

