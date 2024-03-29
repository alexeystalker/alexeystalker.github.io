---
share: true
tags:
 - event-sourcing
 - microservice/saga
---
# Реализация повествований на основе хореографии с помощью порождения событий
Событийная природа [[event-sourcing-pattern|шаблона “порождение событий”]] делает реализацию повествований на основе географии довольно прямолинейной. При обновлении агрегат генерирует событие.  У другого агрегата может быть обработчик, который обновляет его в результате получения события. Фреймворк для порождения событий делает все обработчики идемпотентными.
Проблема использования событий для хореографии повествований состоит в их двойном назначении. В шаблоне “порождение событий” они описывают изменение состояния, но в хореографии повествований должны генерироваться агрегатом, даже если состояние не меняется. Например, если обновление агрегата нарушает бизнес-правило, тот должен сгенерировать событие, чтобы сообщить об ошибке. Если участнику повествования не удалось создать агрегат — вернуть ошибку просто некому.
Учитывая эти трудности, более сложные повествования лучше реализовывать с помощью оркестрации.