---
share: true
tags:
 - gRPC/unary
 - gRPC/client-streaming
 - gRPC/server-streaming
 - gRPC/duplex-streaming
---
# Типы сервисов gRPC
gRPC поддерживает 4 типа вызовов к серверу:
- Унарный
- Поток от сервера
- Поток от клиента
- двухсторонний поток

В отличие от REST API который поддерживает только унарный тип вызовов (клиент отправляет запрос, сервер получает его и возвращает ответ), gRPC поддерживает все 4 типа[^1].
## Унарные вызовы
Клиент инициирует удалённый вызов процедуры с помощью указания имени метода, метаданных и сообщения запроса. Сервер возвращает ответ, содержащий gRPC-статус, сообщение ответа и метаданные. 
```mermaid
sequenceDiagram
participant C as gRPC Client
participant S as gRPC Server
C->>S: Имя метода + сообщение запроса (request message) + метаданные
S->>C: Сообщение ответа (response message) + gRPC-статус + метаданные 
```
## Вызовы с потоком от сервера
Клиент инициирует удалённый вызов процедуры с помощью указания имени метода, метаданных и сообщения запроса. Затем он получает поток от сервера. Сервер возвращает gRPC-cтатус только после того, как все запрошенные данные переданы. Вот как это выглядит на диаграмме.
```mermaid
sequenceDiagram
participant C as gRPC Client
participant S as gRPC Server
C->>S: Имя метода + сообщение запроса (request message) + метаданные
S->>C: Сообщение 1
S->>C: Сообщение 2
S->>C: Сообщение 3 
S->>C: gRPC-статус + метаданные 
```
## Вызовы с потоком от клиента
Клиент инициирует удалённый вызов процедуры с помощью указания имени метода и метаданных. Затем клиент отправляет поток сообщений. Однако, сервер может отправить код статуса и метаданные до отправки всех клиентских сообщений. Диаграмма:
```mermaid
sequenceDiagram
participant C as gRPC Client
participant S as gRPC Server
C->>S: Имя метода + метаданные
C->>S: Сообщение 1
C->>S: Сообщение 2
C->>S: Сообщение 3
S->>C: Сообщение ответа (response message) + gRPC-статус + метаданные
```
## Вызовы с двухсторонним потоковым обменом
Каждый из участников взаимодействия отправляет свои сообщения в потоковом режиме, это может происходить параллельно, что означает, что порядок, в котором отправляются сообщения от клиента и от сервера, отсутствует. Клиент инициирует удалённый вызов процедуры с помощью указания имени метода и метаданных. Затем сервер может немедленно ответить возвратом статуса и метаданных (или же может это сделать, когда клиент отправит все свои сообщения). Диаграмма:
```mermaid
sequenceDiagram
participant C as gRPC Client
participant S as gRPC Server
C->>S: Имя метода + метаданные
S->>C: Сообщение ответа 1
C->>S: Сообщение запроса 1
S->>C: Сообщение ответа 2
S->>C: Сообщение ответа (response message) + gRPC-статус + метаданные
```

[^1]: Также все 4 типа поддерживает SignalR, подробнее про стриминг в SignalR [[ch-6-streaming-in-signalr|здесь]].