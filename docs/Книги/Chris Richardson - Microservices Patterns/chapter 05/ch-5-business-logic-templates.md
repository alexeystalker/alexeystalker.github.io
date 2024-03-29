---
share: true
tags:
 - microservice/domain-model
 - business-logic/patterns
---
# Шаблоны организации бизнес-логики
Вот [[ch-2-hexagonal-architecture|гексагональная архитектура]] типичного сервиса. Ее ядром является *бизнес-логика*, и чаще всего это самая сложная часть сервиса.
Существует два основных шаблона организации бизнес-логики:
- процедурный [[ch-5-transaction-script|"Сценарий транзакции"]]
- объектно-ориентированный [[ch-5-domain-model|"Доменная модель"]]

У шаблона “доменная модель” есть ряд проблем в контексте микросервисной архитектуры. Чтобы разобраться с ними, нужно использовать версию ООП, известную как [[ch-5-domain-driven-design|DDD]].
