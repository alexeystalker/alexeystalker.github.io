---
share: true
tags:
 - microservice/pattern
 - microservice/architecture
 - microservice/chassis
---
# Шаблон "Шасси микросервисов"
![[Pasted image 20211103204415.png]]
Создание сервисов на основе фреймворка или группы фреймворков, которые реализуют общие, не связанные с бизнес-логикой функции, такие как:
- [[circuit-breaker-pattern|предохранитель]]
- [[access-token-pattern|работу с токенами]]
- [[client-side-discovery-pattern|обнаружение сервисов]]
- [[externalized-configuration-pattern|обращение к серверу конфигурации]]
- [[observability-patterns|наблюдаемость]]

Недостатком является то, что отдельное шасси нужно подбирать для каждой платформы отдельно.
## Ссылки
https://microservices.io/patterns/microservice-chassis.html