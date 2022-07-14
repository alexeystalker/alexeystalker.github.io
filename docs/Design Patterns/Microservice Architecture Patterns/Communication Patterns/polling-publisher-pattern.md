---
share: true
tags:
 - microservice/pattern
 - microservice/communication
 - microservice/messaging
 - microservice/transactions
---
# Шаблон "Опрашивающий издатель"
Если приложение использует реляционную БД, сообщения, вставленные в таблицу `OUTBOX` ретранслятор может вычитывать из нее неопубликованные записи обычным SQL запросом типа
```sql
SELECT * FROM OUTBOX ORDERED BY ... ASC
```
Затем публикует полученные сообщения в соответствующие каналы брокера. После успешной (!) публикации сообщения можно удалить:
```sql
DELETE FROM OUTBOX WHERE ID IN (...)
```
## Ссылки
https://microservices.io/patterns/data/polling-publisher.html