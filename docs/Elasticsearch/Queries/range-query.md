---
tags: [Elasticsearch/query, Elasticsearch/term-level]
share: true
---

# Range query
Возвращает документы, которые содержат значения в заданном диапазоне.
Пример:
```json
GET /_search
{
  "query": {
    "range": {
      "age": {
        "gte": 10,
        "lte": 20,
        "boost": 2.0
      }
    }
  }
}
```
## Параметры
`field` (required) - поле, по которому нужно осуществить поиск. Указывается как имя поля запроса, а не как значение поля `field`. Является объектом.
## Параметры `field`
Все -  optional
- `gt` - больше чем
- `gte` - больше или равно чем
- `lt` - меньше чем
- `lte` - меньше или равно чем
- `format` - переопределяет значение формата для даты
- `relation` - как запрос работает с [[range-field|range]] полями. Возможные значения:
	- `INTERSECTS` (default) - диапазон поля пересекается с диапазоном запроса
	- `CONTAINS` - диапазон поля полностью содержит диапазон запроса
	- `WITHIN` - диапазон поля  полностью содержится в диапазоне запроса
- `time_zone` - используется для преобразования дат в запросе к UTC
- `boost` - используется для увеличения или уменьшения показателя релевантности

## Замечания
см. в [документации](https://www.elastic.co/guide/en/elasticsearch/reference/7.8/query-dsl-range-query.html#range-query-notes) %% Если столкнусь, дополнить %%

## Ссылки
https://www.elastic.co/guide/en/elasticsearch/reference/7.8/query-dsl-range-query.html