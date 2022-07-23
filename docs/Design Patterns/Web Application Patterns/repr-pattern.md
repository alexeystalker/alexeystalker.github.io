---
tags: [OOP/pattern, Web/pattern]
share: true
---
# Шаблон "Request-Endpoint-Response"
*REPR* (reaper) - шаблон проектирования, предназначенный для приложений с API.
При реализации паттерна логика разделяется на три компонента:
- *Запрос (request)* - данные, передаваемые конечной точке;
- *Конечная точка (endpoint)* - логика, выполняемая при получении запроса;
- *Ответ (response)* - данные, возвращаемые вызывающему.

Предложен, по видимому, [Стивом "Ардалисом" Смитом](https://github.com/ardalis) 
Для реализации шаблона он же предлагает [фреймворк ApiEndpoints](https://github.com/ardalis/ApiEndpoints)

**NB:** Кажется, в ситуации, когда API представляет собой бэкенд для, например, SPA-приложения, этот шаблон не очень подходит, так как в этом случае конечных точек может быть очень много, и идеология "один файл на конечную точку" порождает очень много файлов.

## Ссылки
[Статья Стива Смита](https://ardalis.com/mvc-controllers-are-dinosaurs-embrace-api-endpoints/)
[Нугет ApiEndpoints](https://www.nuget.org/packages/Ardalis.ApiEndpoints/)
[[library-fastendpoints|Еще один фреймворк FastEndpoints]]