---
tags: [Elasticsearch/aggs]
share: true
---
# Nested aggregation
"Однокорзинная" агрегация для построения агрегаций по вложенным (nested) документам. Необходима, если требуется агрегировать значения полей в nested документах.
Пример из документации:
```json
{
  "query": {
    "match": { "name": "led tv" }
  },
  "aggs": {
    "resellers": {
      "nested": {
        "path": "resellers"
      },
      "aggs": {
        "min_price": { "min": { "field": "resellers.price" } }
      }
    }
  }
}
```

## Ссылки
https://www.elastic.co/guide/en/elasticsearch/reference/7.8/search-aggregations-bucket-nested-aggregation.html
