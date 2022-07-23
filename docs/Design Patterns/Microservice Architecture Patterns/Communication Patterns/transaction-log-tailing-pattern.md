---
share: true
tags:
 - microservice/pattern
 - microservice/communication
 - microservice/messaging
 - microservice/transactions
---
# Шаблон "Отслеживание транзакционного журнала"
Суть в том, что ретранслятор *отслеживает* транзакционный журнал БД. Каждое обновление БД (в том числе [[transactional-outbox-pattern|вставка]] в `OUTBOX` ) представляется в виде записи в журнале транзакций. Можно прочитать этот журнал и опубликовать каждое изменение в качестве сообщения для брокера.
![[Pasted image 20210916210026.png]]
*Анализатор журнала транзакций* читает записи в журнале и каждую подходящую запись преобразует в событие и публикует для брокера. Этот подход годится как для SQL, так и для NoSQL СУБД.
Существует несколько примеров реализации такого подхода
- [Debezium](https://debezium.io/) - проект с открытым исходным кодом, публикует изменения БД для брокера Apache Kafka
- [Linkedin Databus](https://github.com/linkedin/databus) - проект с открытым исходным кодом, анализирует журнал Oracle и публикует изменения в виде событий
- [DynamoDB Streams](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Streams.html) - создает поток упорядоченных по времени изменений в таблицах за последние 24 часа. Приложение может читать эти изменения из потока и, например, публиковать в виде событий
- [Eventuate Tram](https://github.com/eventuate-tram/eventuate-tram-core) - библиотека автора книги для транзакционного обмена сообщениями, использует протокол журнала MySQL, Postgres WAL или просто проверяет изменения в OUTBOX и публикует в Apache Kafka
## Ссылки
https://microservices.io/patterns/data/transaction-log-tailing.html