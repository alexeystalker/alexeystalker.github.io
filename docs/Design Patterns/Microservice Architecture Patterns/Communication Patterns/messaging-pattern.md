---
share: true
tags:
 - microservice/pattern
 - microservice/communication
 - microservice/messaging
---
# Шаблон "Обмен сообщениями"
Сервисы асинхронно коммуницируют путём обмена сообщениями.
Существуют следующие стили асинхронной коммуникации:
+ [[request-response-messaging|Запрос/ответ]] - клиент отправляет запрос и ожидает вскоре получить ответ
+ [[notifications-messaging|Уведомления]] - клиент отправляет уведомление (или управляющую команду), при этом не ожидая ответа.
+ [[request-async-response-messaging|Запрос/асинхронный ответ]] - клиент отправляет запрос, рассчитывая получить ответ "когда-нибудь"
+ [[publish-subscribe-messaging|Публикация/подписка]] - клиент публикует сообщение для нуля или более подписчиков
+ [[publish-async-response-messaging|Публикация/асинхронный ответ]] - клиент публикует запрос к одному или более клиентам, некоторые из них присылают ответ
## Ссылки
https://microservices.io/patterns/communication-style/messaging.html