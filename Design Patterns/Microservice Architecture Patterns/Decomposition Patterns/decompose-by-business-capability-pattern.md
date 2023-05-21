---
share: true
tags:
 - microservice/pattern
 - microservice/decomposition
---
# Шаблон "Разбиение по бизнес-возможностям"
Концепция "бизнес-возможности" применяется в моделировании бизнес-структур и обозначает то, из чего бизнес генерирует прибыль. Бизнес-возможности организации описывают то, *чем* она является. Обычно бизнес-возможности стабильны, в отличие от того, *как* организация ведет свой бизнес.
Бизнес-возможности организации определяются путём анализа ее целей, структуры и бизнес-процессов.
Бизнес-возможность часто сосредоточена на определенном бизнес-объекте. Во многих случаях возможность можно разбить на подвозможности.
Пример разбиения:
![[Pasted image 20210831184449.png]]
Ключевое преимущество шаблона в том, что из-за стабильности возможностей итоговая архитектура тоже будет относительно стабильной.

## Ссылки
https://microservices.io/patterns/decomposition/decompose-by-business-capability.html