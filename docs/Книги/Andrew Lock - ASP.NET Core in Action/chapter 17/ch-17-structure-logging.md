---
share: true
tags:
 - NET/logging
---
# Структурное журналирование: создание полезных сообщений журналов с возможностью поиска
Предположим, у нас есть приложение с подключённым журналированием в файл. Тогда в файле при записи сообщения они будут храниться в виде
```log
warn: RecipeApplication.Controllers.RecipeCOntroller[12]
      Could not find recipe with id 3245
```
Однако это никак не помогает нам понять, почему случилось то, что случилось? Происходило ли это с другими рецептами? Как часто? И тому подобные вопросы.
Можно использовать инструменты для поиску по тексту, но значительно удобнее, если сообщение хранится в [[structured-logging|структурированном виде]]. Например, сообщение выше в структурированном виде выглядило бы так (используем формат JSON):
```json
{
	"eventLevel": "Warning",
	"category": "RecipeApplication.Controllers.RecipeController",
	"eventId": 12,
	"messageTemplate": "Could not find recipe with {recipeId}",
	"message": "Could not find recipe with id 3245",
	"recipeId": "3245"
}
```
Теперь сообщение записано в формате, который позволит легко искать определенные записи журнала.
> [!note] Примечание
> Формат записи сообщения может различаться в зависимости от используемого поставщика журналирования. Ключевым моментом является использование пар “ключ-значение” для сохранения свойств.

Для добавления структурного журналирования требуется поставщик, который может создавать и хранить структурированные сообщения. Например, у [[library-serilog|Serilog]] есть получатель данных для записи в Elasticsearch.
Elasticsearch — мощный инструмент, и часто используется в промышленном окружении для хранения журналов, однако он весьма непрост в настройке. Есть и другие, более удобные варианты, например [[datalust-seq|Seq]].
#### [[ch-17-adding-a-structured-logging-provider|Добавление поставщика структурного журналирования в приложение]]

---
Также при использовании структурного журналирования может быть очень полезен такой инструмент как *области журналирования (scopes)*
#### [[ch-17-using-scopes-to-add-additional-properties-to-logs|Использование областей журналирования для добавления дополнительных свойств в сообщения журнала]]