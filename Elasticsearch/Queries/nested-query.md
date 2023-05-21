---
tags: [Elasticsearch/query, Elasticsearch/joining]
share: true
---
# Nested query
Для построения запроса по nested документам нужно сделать запрос типа `nested`, nested-поле указать в параметре `path`, следующим параметрам указать query по вложенному документу, причем поля для поиска указываются в виде `%path%.%field%`. Например, [[exists-query|Exists]] запрос для вложенного поля:
```json
{
  "query": {
    "nested": {
      "path": "leasing",
      "query": {
        "exists": {
          "field": "leasing.leasing-id"
        }
      }
    }
  }
}
```

## Ссылки
https://stackoverflow.com/questions/41491982/elastic-exists-query-for-nested-documents (в примере по ссылке почему-то `exists` оборачивают в `bool`, это работает но выглядит оверкиллом)
https://www.elastic.co/guide/en/elasticsearch/reference/7.8/query-dsl-nested-query.html
