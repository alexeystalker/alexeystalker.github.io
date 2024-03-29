---
title: gRPC
aliases:
 - gRPC
tags: [protocol,gRPC]
share: true
---
# gRPC (Remote Procedure Call от Google)
gRPC — фреймворк для написания многоязычных клиентов и серверов 
Представляет собой двоичный протокол на основе сообщений.
В gRPC API описывается с помощью IDL на основе Protocol Buffers — многоязычного механизма сериализации структурированных данных от Google. Компилятор Protocol Buffer генерирует клиентские заглушки и серверные каркасы. Клиент и сервер обмениваются сообщениями в формате [[../../Книги/Anthony Giretti - Beginning gRPC with ASP.NET Core 6/chapter 04/ch-4-protobufs|Protocol Buffers]], используя [[../Протоколы/http-2|HTTP/2]].
gRPC API поддерживает следующие подвиды [[../../Microservices/Messages/request-response-messaging|коммуникации “запрос-ответ”]]:
- [[./grpc-unary-call|унарный вызов]] — одно сообщение запроса, одно сообщение ответа;
- [[./grpc-client-streaming-call|клиентский поток]] — клиент отправляет поток сообщений одной структуры, сервер обрабатывает его и возвращает результат;
- [[./grpc-server-streaming-call|серверный поток]] — в ответ на запрос клиента возвращается поток сообщений одной структуры;
- [[./grpc-duplex-streaming-call|двунаправленный поток]] — клиент отправляет поток сообщений одной структуры, сервер, не дожидаясь завершения отправки клиентского потока, отправляет поток сообщений на клиент.

## Преимущества
+ Позволяет легко спроектировать API с богатым набором операций обновления
+ Имеет эффективный компактный механизм, что особенно явно проявляется при обмене крупными сообщениями
+ Поддержка двунаправленных потоков делает возможными стили взаимодействия на основе RPI и обмена сообщениями
+ Позволяет сохранять совместимость между клиентами и сервисами, написанными на разных языках
## Недостатки
- Процесс работы с API, основанном на gRPC оказывается для Javascript клиентов более трудоёмким, чем с API, основанным на REST/JSON
- Старые брандмауэры не поддерживают HTTP/2

## Ссылки
https://grpc.io/
[[../../Книги/Anthony Giretti - Beginning gRPC with ASP.NET Core 6/book-beginning-grpc-with-asp-net-core-6|Конспект книжки “Начала использования gRPC с ASP.NET Core 6”]]