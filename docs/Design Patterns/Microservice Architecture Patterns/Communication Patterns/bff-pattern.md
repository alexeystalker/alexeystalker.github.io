---
share: true
tags:
 - microservice/pattern
 - microservice/communication
 - microservice/api
---
# Шаблон "Бэкенды для фронтендов" (BFF)
`Backends for Frontends`
Разновидность шаблона [[api-gateway-pattern|API-шлюз]], идея которого заключается в том, что каждый API-модуль становится отдельным API-шлюзом
![[Pasted image 20211016113601.png]]
Преимущества:
- API-модули изолированы друг от друга
- возможность независимого масштабирования
- уменьшение времени запуска
## Ссылки
https://microservices.io/patterns/apigateway.html