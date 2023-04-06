---
share: true
tags:
 - gRPC
---
# gRPC — Общие понятия
Проект gRPC зародился в 2000-е, исходный код был открыт в 2014. gRPC использует Protocol Buffers (Protobuf) в качестве IDL, сообщения сериализуются в бинарный формат и передаются посредством протокола [[http-2|HTTP/2]]. gRPC даёт богатый выбор языков разработки, среди которых
- Java
- C
- C++
- Node.js (JavaScript/TypeScript)
- Python
- Ruby
- GO
- Dart
- PHP
- C\#

Как и любой другой RPC, gRPC определяет одну или несколько процедур, которые могут быть вызваны удалённо. На серверной стороне сервер предоставляет функции для обработки клиентских вызовов. Клиент может быть реализован на любом из языков, поддерживающих те же gRPC методы, что и сервер.
#### [[ch-3-protocol-buffers|Protocol Buffers]]
#### [[ch-3-grpc-channel|Канал gRPC]]
#### [[ch-3-types-of-grpc-services|Типы сервисов gRPC]]
#### [[ch-3-trailers|Трейлеры]]
#### [[ch-3-grpc-status|gRPC-статус]]
#### [[ch-3-deadline-and-cancellation|Дедлайн и отмена]]
#### [[ch-3-grpc-requests-and-responses-over-http-2|Запросы и ответы gRPC на основе HTTP/2]]
