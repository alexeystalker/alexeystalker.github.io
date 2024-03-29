---
share: true
tags:
 - microservice/pattern
 - microservice/data-management
 - DDD/domain-event
 - event-sourcing
---
# Шаблон “Порождение событий”
Сохраняет [[aggregate-pattern|агрегат]] в виде последовательности [[domain-event-pattern|доменных событий]], которые представляют изменение состояния. Приложение воссоздает текущее состояние, воспроизводя записанные события.
События сохраняются в *хранилище событий* (event store), у которого есть API для добавления и получения событий сущности. Также хранилище событий выполняет роль брокера сообщений — у него есть API для подписки на события.
Если для какой-то сущности событий очень много, можно использовать *снэпшоты* состояния сущности. Тогда для восстановления текущего состояния нужно воспроизвести события только до последнего снэпшота.
## Создание и обновление
### Шаги создания агрегата
1. Создание экземпляра корня агрегата конструктором по умолчанию.
2. Вызов `process()` для генерации событий создания.
3. Обновление агрегата путём применения созданых событий методом `apply()`.
4. Сохранение событий в хранилище событий.
### Шаги обновления агрегата
1. Загрузка событий из хранилища событий.
2. Создание экземпляра корня агрегата конструктором по умолчанию.
3. Обновление агрегата путём применения загруженных событий методом `apply()`
4. Вызов `process()` для генерации новых событий.
5. Обновление агрегата путём применения новых событий методом `apply()`.
6. Сохранение событий в хранилище событий.
## Преимущества
+ Надежная публикация доменных событий
+ Сохранение истории изменений агрегата
+ Отсутствие большинства проблем, связанных с объектно-реляционным разрывом
+ Машина времени для разработчиков
## Недостатки
- Другая модель программирования с высоким порогом вхождения
- Сложность приложения, основанного на обмене сообщениями
- Меняющиеся события могут создать проблемы
- Усложняется удаление данных
- Трудности при обращении к хранилищу событий

## Ссылки
[[ch-6-event-sourcing-benefits|Подробнее о преимуществах]]
[[ch-6-event-sourcing-drawbacks|Подробнее о недостатках]]
https://microservices.io/patterns/data/event-sourcing.html