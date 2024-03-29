---
share: true
tags:
 - microservice/observability
---
# Проектирование наблюдаемых сервисов
Чтобы сделать сервисы более простыми для управления и диагностики, разработчик обязан использовать определенные шаблоны проектирования.
![[Pasted image 20211101195345.png]]
 Эти шаблоны предоставляют информацию о поведении и работоспособности сервиса, позволяют системе мониторинга отслеживать и визуализировать состояние сервиса и генерировать оповещение при возникновении проблемы.
 - [[health-check-api-pattern|API проверки работоспособности]] — предоставляет конечную точку, которая возвращает данные о работоспособности сервиса.
 - [[log-aggregation-pattern|Агрегация журналов]] — ведет журналы активности сервисов и сохраняет их на центральном журнальном сервере с поддержкой поиска и оповещений
 - [[distributed-tracing-pattern|Распределенная трассировка]] — назначает каждому внешнему запросу уникальный идентификатор и отслеживает запросы по мере перемещения между сервисами
 - [[exception-tracking-pattern|Отслеживание исключений]] — за исключениями следит отдельный сервис, который избавляется от дубликатов, оповещает разработчиков и отслеживает обработку каждого исключения
 - [[application-metrics-pattern|Метрики приложения]] — сервисы собирают метрики, такие как счётчики и оценочные показатели и делают их доступными серверу метрик.
 - [[audit-logging-pattern|Ведение журнала аудита]] — ведет журнал действий пользователей.

