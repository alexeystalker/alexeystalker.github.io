---
tags: [Elasticsearch/query, Elasticsearch/term-level]
share: true
---
# IDs query
Возвращает документы, соответствующие переданным Id (Id хранится в поле `_id`).
```json
GET /_search
{
  "query": {
    "ids" : {
      "values" : ["1", "4", "100"]
    }
  }
}
```
Я думаю, может быть особенно полезен для запросов к подчинённым документам, например, [[nested-query|nested]] с [[inner-hits|inner_hits]].
Пусть есть документы с nested документом в поле `nstd`, где `nstd.text` - анализируемое поле, а `nstd.id` - некий идентификатор. Тогда найдём не более 5 вложенных документов с полем `nstd.text`, удовлетворяющих [[match-query|match запросу]]:
```json
{
  "query": {
    "bool": {
      "must": [
          {
              "ids": {
                  "values": [
                      "1",
                      "10",
                      "106",
                      "774",
                      "377"
                      ]
              }
          },
        {
          "nested": {
            "path": "nstd",
            "query": {
              "match": {
                "nstd.text": "москва"
              }
            },
            "inner_hits": {
              "name": "nested_with_text",
              "_source": false,
              "docvalue_fields": [
                "brnch.id"
              ],
              "size": 5
            }
          }
        }
      ]
    }
  },
  "_source": false
}
```

## Ссылки
https://www.elastic.co/guide/en/elasticsearch/reference/7.8/query-dsl-ids-query.html