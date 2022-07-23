---
share: true
tags:
 - microservice/pattern
 - microservice/observability
---
# Шаблон "Отслеживание исключений"
Сервис сообщает об исключениях центральному сервису, который их дедуплицирует, отслеживает их исправление и генерирует оповещения.
![[Pasted image 20211103194708.png]]
Существует некоторое количество сервисов отслеживания исключений, например [Honeybadger](https://www.honeybadger.io/) или [[sentry-io|Sentry]]
## Ссылки
https://microservices.io/patterns/observability/exception-tracking.html