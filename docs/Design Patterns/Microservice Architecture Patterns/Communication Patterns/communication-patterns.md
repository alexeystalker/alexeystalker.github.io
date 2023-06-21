---
share: true
tags:
 - microservice/patterns
 - microservice/communication
---
# Шаблоны взаимодействия
![[Pasted image 20210826151226.png]]
Шаблоны взаимодействия делятся на пять групп
+ *Стиль взаимодействия* между сервисами
	+ [[rpi-pattern|Удаленный вызов процедур]]
	+ [[messaging-pattern|Обмен сообщениями]]
	+ [[idempotent-consumer-pattern|Идемпотентный потребитель]]
+ *Обнаружение* — Каким образом клиент сервиса узнает, куда отправить запрос?
	+ [[service-registry-pattern|Реестр]]
	+ [[self-registration-pattern|Саморегистрация]]
	+ [[third-party-registration-pattern|Сторонняя регистрация]]
	+ [[client-side-discovery-pattern|Обнаружение на стороне клиента]]
	+ [[server-side-discovery-pattern|Обнаружение на стороне сервера]]
+ *Надежность* — Как обеспечить надежное взаимодействие с учетом того, что некоторые сервисы могут быть недоступны?
	+ [[circuit-breaker-pattern|Предохранитель]]
	+ [[retry-pattern|Повтор]]
+ *Транзакционный обмен сообщениями* — как интегрировать отправку сообщений и публикацию событий с транзакциями БД?
	+ [[transactional-outbox-pattern|Outbox - публикация событий]]
	+ [[polling-publisher-pattern|Опрашивающий издатель]]
	+ [[transaction-log-tailing-pattern|Отслеживание транзакционного журнала]]
+ *Внешний API* — Каким образом внешние клиенты приложения взаимодействуют с сервисами?
	+ [[api-gateway-pattern|API-шлюз]]
	+ [[bff-pattern|Бэкенды для фронтендов(BFF)]]
