---
share: true
tags:
 - microservice/pattern
 - microservice/communication
 - microservice/messaging
 - microservice/transactions
---
# Шаблон "Опрашивающий издатель"
Пусть приложение использует реляционную БД, и [[transactional-outbox-pattern|хранит сообщения]] в таблице `OUTBOX`. Ретранслятор может вычитывать из этой таблицы неопубликованные записи обычным SQL запросом типа
```sql
SELECT * FROM OUTBOX ORDERED BY ... ASC
```
Затем он публикует полученные сообщения в соответствующие каналы брокера. После успешной (!) публикации сообщения можно удалить:
```sql
DELETE FROM OUTBOX WHERE ID IN (...)
```
## Ссылки
https://microservices.io/patterns/data/polling-publisher.html