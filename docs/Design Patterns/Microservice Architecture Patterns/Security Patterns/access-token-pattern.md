---
share: true
tags:
 - microservice/pattern
 - microservice/security
---
# Шаблон “Токен доступа”
[[api-gateway-pattern|API-шлюз]] аутентифицирует клиента и передает токен с информацией о пользователе, включая его идентификатор и роли, сервисам, к которым тот обращается. Если нижележащие сервисы обращаются к третим сервисам, они передают токен в своих запросах.
Вариантом такого токена может выступать [[json-web-token|JSON Web Token]]
## Ссылки
https://microservices.io/patterns/security/access-token.html