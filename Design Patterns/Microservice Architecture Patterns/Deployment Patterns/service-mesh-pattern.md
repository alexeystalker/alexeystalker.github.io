---
share: true
tags:
 - microservice/pattern
 - microservice/deployment
 - microservice/communication
---
# Шаблон "Сеть сервисов"
Пропускает весь трафик между сервисами через сетевой слой, реализующий такие функции как [[circuit-breaker-pattern|предохранители]], [[distributed-tracing-pattern|распределенная трассировка]], [[server-side-discovery-pattern|обнаружение сервисов]], балансирование нагрузки и маршрутизация трафика.
Некоторые реализации сети сервисов:
- [Istio](https://istio.io/)
- [Linkerd](https://linkerd.io/)
## Ссылки
https://microservices.io/patterns/deployment/service-mesh.html
