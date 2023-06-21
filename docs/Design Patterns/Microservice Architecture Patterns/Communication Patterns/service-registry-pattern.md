---
share: true
tags:
 - microservice/pattern
 - microservice/communication
 - microservice/discovery
---
# Шаблон “Реестр сервисов”
БД, хранящая актуальные адреса сервисов. Сервисы регистрируются ([[self-registration-pattern|сами]] или не сами) в реестре при старте и удаляются из реестра при выключении. Также реестр может обращаться к Healthcheck API  для проверки доступности экземпляров.
## Ссылки
https://microservices.io/patterns/service-registry.html

