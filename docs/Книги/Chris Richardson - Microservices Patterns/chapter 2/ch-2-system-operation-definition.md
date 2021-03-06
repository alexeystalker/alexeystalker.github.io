---
share: true
tags: [microservice,книга,конспект]
---
# Определение системных операций
Это первый шаг в проектировании архитектуры приложения. За отправную точку берутся требования к приложению, включая пользовательские истории (user stories) и связанные с ними сценарии использования. 
![[Pasted image 20210830192414.png]]
Процесс состоит из двух шагов.
1. Создаем обобщенную (или абстрактную) доменную модель, состоящую из ключевых классов, которые представляют словарь для описания системных операций. Используем, в основном, имена существительные, взятые из пользовательских историй. Также доменную модель можно определить с помощью методики [[ch-5-event-storm|Событийный штурм]].![[Pasted image 20210830195216.png]]
2. Описываем системные операции, а также их поведение с точки зрения доменной модели. Используем, в основном, глаголы. Системная операция может создавать, обновлять или удалять доменные объекты, а также устанавливать или разрушать связи между ними.

Подробнее рассмотрим определение системной операции.
Все системные операции делятся на два вида: 
- *команды* - системные операции для создания, обновления и удаления данных. У команды есть спецификация, состоящая из:
	- параметров
	- возвращаемого значения
	- предварительных условий (должны выполняться в момент вызова команды)
	- окончательных условий (должны выполняться после вызова команды)
- *запросы* - системные операции для чтения (запрашивания) данных.

Пример спецификации для команды createOrder():

|spec item|value|
|--|--|
|имя и параметры|createOrder(ID клиента, способ оплаты, адрес доставки, время доставки, ID ресторана, позиции заказа)|
|возвращает|orderId|
|Предварительные условия|Клиент сущетсвует и может размещать заказы; позиции заказа соответствуют пунктам меню ресторана; адрес и время доставки выполнимы для ресторана|
|Окончательные условия|Банковская карта клиента позволила снять сумму заказа; Заказ был создан в состоянии PENDING_ACCEPTANCE|

*Запросы* обычно наполняют пользовательский интерфейс информацией, необходимой для принятия решений.
