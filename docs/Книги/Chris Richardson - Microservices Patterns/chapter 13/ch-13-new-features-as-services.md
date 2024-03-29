---
share: true
tags:
 - microservice/transition
---
# Реализация новых возможностей в виде сервисов
[[law-of-holes|Закон ямы]] гласит “если вы оказались в яме, перестаньте копать”. Иными словами, если если монолитный проект уже большой и сложный, пора перестать добавлять в него усложняющие новые функции. Вместо этого новые функции следует реализовывать в виде сервисов.
## Интеграция нового сервиса с монолитом
Архитектура приложения после реализации новой возможности в виде сервиса.
![[Pasted image 20211106150706.png]]
Помимо монолита и нового сервиса, мы видим два новых компонента:
- [[api-gateway-pattern|API-шлюз]] направляет запросы новой функциональности к новым сервисам, а старой — к монолиту.
- *Интеграционный связующий код* — интегрирует сервисы в монолит. Позволяет сервису обращаться к данным, принадлежащим монолиту.
## Когда новую функцию следует реализовывать в виде сервиса
В идеале каждая новая функция должна быть реализована в удушающем приложении, а не в монолите, для чего создается новый или дополняется уже существующий сервис. К сожалению, такое возможно не всегда.
Возможность может оказаться слишком мелкой для того, чтобы быть значимым сервисом. Или же новая функциональность слишком тесно связана с кодом монолита, и если реализовать ее в виде сервиса, это выльется в излишнее межпроцессное взаимодействие.
