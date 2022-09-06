---
share: true
tags:
 - microservice/pattern
 - microservice/communication
 - microservice/discovery
---
# Шаблон "Обнаружение на клиентской стороне"
Клиент извлекает из [[service-registry-pattern|реестра]] список доступных экземпляров сервиса (для снижения нагрузки может быть применено кэширование) и использует алгоритм балансирования нагрузки, чтобы выбрать конкретный экземпляр и отправить ему запрос.
## Ссылки
https://microservices.io/patterns/client-side-discovery.html
