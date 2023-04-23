---
share: true
tags:
 - gRPC
 - HTTP-2
---
# Увеличиваем максимальное число соединений HTTP/2
По умолчанию Kestrel и другие сервера позволяют выполнять одновременно 100 запросов на одно соединение HTTP/2. Канал gRPC использует одно соединение HTTP/2, при этом, если возникает необходимость в создании более чем ста активных запросов, превышающие это число запросы будут поставлены в очередь. Существует возможность создания дополнительных соединений в случае, когда это необходимо. Вот как включить эту возможность (отметим, что эту настройку можно комбинировать с другими):
```csharp
var handler = new SocketsHttpHandler
{
    KeepAlivePingDelay = TimeSpan.FromSeconds(15),
    PooledConnectionIdleTimeout = TimeSpan.FromMinutes(5),
    KeepAlivePingTimeout = TimeSpan.FromSeconds(5),
    EnableMultipleHttp2Connections = true //<--
};
```
