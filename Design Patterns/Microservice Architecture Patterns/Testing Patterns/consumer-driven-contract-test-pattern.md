---
share: true
tags:
 - microservice/pattern
 - microservice/testing
---
# Шаблон "тестирование контрактов с расчётом на потребителя"
Идея в том, чтобы потребители писали тесты, проверяющие контракт сервиса (provider) на соответствие нуждам потребителей (consumers). Взаимодействие между потребителем и провайдером определяется набором примеров сообщений, которые называются *контрактами*.
![[Pasted image 20211022200513.png]]
## Ссылки
https://microservices.io/patterns/testing/service-integration-contract-test.html