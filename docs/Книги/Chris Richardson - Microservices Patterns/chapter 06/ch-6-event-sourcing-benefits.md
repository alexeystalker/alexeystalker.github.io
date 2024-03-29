---
share: true
tags:
 - event-sourcing
---
# Преимущества порождения событий
## Надежная публикация доменных событий
Основное преимущество порождения событий — надежная публикация уведомлений о каждом изменении состояния агрегата. Также событие может хранить ID пользователя, что позволяет вести журнал аудита. Поток событий можно использовать для других задач: уведомление пользователей, аналитика, мониторинг, уведомление других приложений
## Сохранение истории изменений агрегата
Можно легко определять состояние агрегата в любой прошлый момент времени — достаточно свернуть события, произошедшие до этого момента.
## Отсутствие большинства проблем, связанных с объектно-реляционным разрывом
Порождение событий основывается скорее на их постоянном хранении, чем на агрегации.События обычно имеют простую структуру, которую легко сериализовать. [[ch-6-snapshots-improve-performance|Как упоминалось]], сервис может сделать снимок сложного агрегата. Это вводит новый уровень опосредованности между агрегатом и его сериализованным представлением.
## Машина времени для разработчиков
Шаблон “порождение событий” хранит историю всего, что происходило на протяжении жизненного цикла приложения, что позволяет разработчикам реализовывать непредвиденные требования даже над данными, возникшими в прошлом, как бы “перемещаться в прошлое”,