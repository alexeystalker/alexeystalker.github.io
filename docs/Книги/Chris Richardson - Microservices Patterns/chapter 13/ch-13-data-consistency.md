---
share: true
tags: [microservice]
---
# Обеспечение согласованности данных между сервисом и монолитом
Предположим, что мы извлекли из монолита сервис `Kitchen`. Из-за этого придется перевести в монолитном коде такие операции как `createOrder()` и `cancelOrder()` на [[saga-pattern|повествования]], чтобы сущности `Ticket` и `Order` остались согласованными.
Однако участие монолита в повествованиях не всегда проходит гладко. Например, повествования должны использовать компенсирующие транзакции для отмены изменений, однако, внедрение компенсирующих транзакций в монолит потребует многочисленных изменений и большого количества времени. Также монолиту, возможно, придется реализовать [[ch-4-lack-of-isolation#Контрмеры|контрмеры]], чтобы справиться с недостаточной изоляцией между повествованиями.
Однако, чаще всего можно спланировать последовательность извлечения сервисов так, чтобы транзакции монолита не было нужды делать [[saga-compensating|компенсируемыми]].
## Трудности изменения монолита для поддержки компенсируемых транзакций
Рассмотрим проблему с компенсирующими транзакциями, которую необходимо решить при извлечении сервиса `Kitchen`. Это подразумевает разделение сущности `Order` и создание в сервисе `Kitchen` сущности `Ticket` и изменение команд в монолите, включая `createOrder()`.
Монолит реализует `createOrder()` в виде одной ACID-транзакции:
1. Проверить детали заказа.
2. Убедиться в том, что клиент может размещать заказы.
3. Авторизовать банковскую карту клиента.
4. Создать заказ.

Эту транзакцию нужно заменить повествованием:
1. В монолите
	1. Создать заказ с состоянием `APPROVAL_PENDING`
	2. Убедиться в том, что клиент может размещать заказы
2. В сервисе `Kitchen`
	1. Проверить детали заказа
	2. Создать заявку с состоянием `CREATE_PENDING`
3. В монолите
	1. Авторизовать карту клиента
	2. изменить состояние заказа на `APPROVED`
4. В сервисе `Kitchen`
	1. Изменить состояние заявки на `AWAITING_ACCEPTANCE`

Сложность реализации этого повествования состоит в том, что первый шаг, на котором создается заказ, должен быть компенсируемым. Сущность `Order` должна поддерживать [[semantic-lock-countermeasure|семантическую блокировку]], сигнализирующую о том, что заказ находится в процессе создания.
Это может потребовать масштабной модификации монолита.
## Повествования не всегда требуют от монолита поддержки компенсируемых транзакций
Поддержка компенсируемых транзакций нужна только в том случае, когда последующие транзакции монолита могут завершиться неудачно. Если же все транзакции монолита являются либо [[saga-turning-point|поворотными]], либо [[saga-repeatable|повторяемыми]], ему не нужно будет ничего компенсировать, и нужно будет внести только небольшие изменения.
Представим, что вместо `Kitchen` мы извлекаем сервис `Order`. Тогда повествование `createOrder()` будет таким:
1. Сервис `Order`
	1. Создать заказ с состоянием `APPROVAL_PENDING`
2. Монолит
	1. Убедиться в том, что клиент может размещать заказы
	2. Проверить детали заказа и создать заявку
	3. Авторизовать банковскую карту клиента
3. Сервис `Order`
	1. Изменить состояние заказа на `APPROVED`

Здесь транзакция монолита является поворотной.
## Планирование извлечения сервисов, чтобы избежать реализации компенсирующих транзакций в монолите
Как мы видели, извлечение `Kitchen` требует реализации в монолите компенсирующих транзакций, а извлечение `Order` — нет. Следовательно, порядок извлечения сервисов имеет значение. Можно сделать так, чтобы все транзакции в монолите были либо поворотными, либо повторяемыми. Например, если извлечь после сервиса `Order` сервис `Consumer`, то это упростит извлечение `Kitchen`.
После извлечения `Consumer` команда `createOrder()` имеет вид:
1. Сервис `Order`
	1. Создать заказ с состоянием `APPROVAL_PENDING`
2. Сервис `Consumer`
	1. Убедиться в том, что клиент имеет право размещать заказы
3. Монолит
	1. Проверить детали заказа и создать заявку
	2. авторизовать карту клиента
4. Сервис `Order`
	1. Изменить состояние заказа на `APPROVED`

Вслед за `Consumer` извлекаем сервис `Kitchen`. Теперь:
1. Сервис `Order` — создать заказ с состоянием `APPROVAL_PENDING`
2. Сервис `Consumer` — убедиться в том, что клиент может размещать заказы.
3. Сервис `Kitchen` — проверить детали заказа и создать заявку с состоянием `PENDING`
4. Монолит — авторизовать банковскую карту клиента
5. Сервис `Kitchen` — изменить состояние заявки на `APPROVED`
6. Сервис `Order` — изменить состояние заявки на `APPROVED`

Здесь транзакция монолита остается поворотной.
Если теперь продолжить рефакторинг монолита, и извлечь сервис `Accounting`, то команда `createOrder()` будет использовать такое повествование:
1. Сервис `Order` — создать заказ с состоянием `APPROVAL_PENDING`
2. Сервис `Consumer` — убедиться в том, что клиент может размещать заказы.
3. Сервис `Kitchen` — проверить детали заказа и создать заявку с состоянием `PENDING`
4. Сервис `Accounting` — авторизовать банковскую карту клиента
5. Сервис `Kitchen` — изменить состояние заявки на `APPROVED`
6. Сервис `Order` — изменить состояние заявки на `APPROVED`
