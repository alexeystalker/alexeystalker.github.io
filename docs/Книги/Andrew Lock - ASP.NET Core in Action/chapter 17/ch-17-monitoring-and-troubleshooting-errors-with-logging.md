---
share: true
tags:
 - NET/logging
---
# Мониторинг и устранение ошибок с помощью журналирования
Журналирование — это процесс записи событий или действий в приложении, например, в консоль, файл, журнал событий Windows и т.д.
Ранее существовала проблема с журналированием, заключавшаяся в том, что каждая библиотека и фреймворк генерировали журналы в несколько разном формате (если вообще это делали).
Теперь у ASP.NET Core есть новый обобщённый интерфейс журналирования. Он используется в самом коде ASP.NET Core, а также сторонними библиотеками.
#### [[ch-17-using-logging-effectively-in-production|Эффективное использование журналирования в промышленном приложении]]
#### [[ch-17-adding-log-messages-to-app|Добавление сообщений журнала в приложение]]
#### [[ch-17-controlling-where-logs-are-written|Контроль места записи журналов с помощью поставщиков журналирования]]
#### [[ch-17-changing-log-verbosity-with-filtering|Изменение избыточности сообщений журналов с помощью фильтрации]]
#### [[ch-17-structure-logging|Структурное журналирование: создание полезных сообщений журналов с возможностью поиска]]