---
share: true
tags:
 - microservice/pattern
---
# Шаблон "Заготовка сервиса"
Альтернатива шаблону [[microservice-chassis-pattern|шасси микросервисов]]. Создается общая заготовка для сервиса со всем необходимым для реализации сервиса кодом. Обычно такой код охватывает следубщие функции:
- [[circuit-breaker-pattern|предохранитель]]
- [[access-token-pattern|работу с токенами]]
- [[client-side-discovery-pattern|обнаружение сервисов]]
- [[externalized-configuration-pattern|обращение к серверу конфигурации]]
- [[observability-patterns|наблюдаемость]]

## Ссылки
https://microservices.io/patterns/service-template.html

