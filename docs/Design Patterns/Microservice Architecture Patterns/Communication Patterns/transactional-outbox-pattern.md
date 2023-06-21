---
aliases:
 - transactional outbox
 - outbox
share: true
tags:
 - microservice/pattern
 - microservice/communication
 - microservice/messaging
 - microservice/transactions
---
# Шаблон “публикация событий” или Outbox
![[Pasted image 20210916203747.png]]
Предположим, что у приложения есть реляционная БД. Используем специальную таблицу `OUTBOX` для вставки сообщений в рамках транзакции бизнес-логики.
Далее *ретранслятор* (это компонент сервиса) вычитывает сообщения из этой таблицы и отдает брокеру.
Если у нас NoSQL БД, добавляем поле со списком сообщений, которые нужно отправить, к каждому бизнес-объекту, порождающему сообщения. Это атомарная операция. Трудностью тут будет эффективный поиск сообщений, которые требуется опубликовать. См. также шаблоны [[polling-publisher-pattern|"Опрашивающий издатель"]] и [[transaction-log-tailing-pattern|"Отслеживание транзакционного журнала"]].
## Ссылки
https://microservices.io/patterns/data/transactional-outbox.html
