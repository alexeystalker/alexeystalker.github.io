---
share: true
tags:
 - microservice/pattern
 - microservice/request
 - microservice/api-composition
---
# Шаблон "Объединение API"
Чтобы выполнить запрос, мы обращаемся к разным сервисам и объединяем результаты.
![[Pasted image 20211004193422.png]]
Структура шаблона предусмативает два типа участников:
- *API-композитор* - реализует операцию запроса, обращаясь к сервис-провайдерам
- *сервис-провайдер* - сервис, которому принадлежат данные, возвращаемые запросом.
## Преимущество
Преимущество этого шаблона - в простоте реализации.
## Недостатки
- дополнительные накладные расходы
- риск снижения доступности
- нехватка транзакционной согласованности данных

## Ссылки
https://microservices.io/patterns/data/api-composition.html