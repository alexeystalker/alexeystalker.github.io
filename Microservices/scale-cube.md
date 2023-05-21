---
tags: [microservice]
share: true
---
# Куб масштабирования
Масштабировать (scale) приложение можно тремя способами, по **трём осям куба**.
## Ось X - экземпляры
Часто применяют в монолитах. Масштабируя по X - просто увеличивают число экземпляров (реплик)
## Ось Y - функциональная декомпозиция
код разбивается на сервисы по выполняемым функциям / зонам ответственности
## Ось Z - секционирование данных
Каждая реплика обрабатывает свою часть данных. Деление по некоему предикату, например, по первому символу в userId. Реплики при этом идентичны
## Ссылки
https://microservices.io/articles/scalecube.html