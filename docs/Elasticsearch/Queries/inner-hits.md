---
tags: [Elasticsearch/query, Elasticsearch/joining]
share: true
---
# inner_hits parameter
В случае использования `parent-child` и [[nested-query|nested]] связей может возникнуть необходимость получения связанных документов, удовлетворяющих запросу.
Для этого используется параметр `inner_hits` в запросе или фильтре.
```json
POST test/_search
{
  "query": {
    "nested": {
      "path": "comments",
      "query": {
        "match": { "comments.number": 2 }
      },
      "inner_hits": {}
    }
  }
}
```
При таком запросе будут возвращены nested документы, соответствующие запросу. Возвращаются они внутри ключа `hits` в ключе `inner_hits`:
```json
{
  ...,
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 1.0,
    "hits": [
      {
        "_index": "test",
        "_type": "_doc",
        "_id": "1",
        "_score": 1.0,
        "_source": ...,
        "inner_hits": {
          "comments": { 
            "hits": {
              "total": {
                "value": 1,
                "relation": "eq"
              },
              "max_score": 1.0,
              "hits": [
                {
                  "_index": "test",
                  "_type": "_doc",
                  "_id": "1",
                  "_nested": {
                    "field": "comments",
                    "offset": 1
                  },
                  "_score": 1.0,
                  "_source": {
                    "author": "nik9000",
                    "number": 2
                  }
                }
              ]
            }
          }
        }
      }
    ]
  }
}
```
## Параметры
- `from` - оффсет, с которого начинаем получать `inner_hits`. *(NB пока не очень понимаю, что это)*
- `size` - максимальное число внутренних хитов в одном хите. По умолчанию 3;
- `sort` - каким образом сортировать внутренние хиты. По умолчанию используется `score`;
- `name` - имя для определенных внутренних хитов. Полезно, когда у тебя в запросе несколько подзапросов с разными `inner_hits`;

Также поддерживаются документные фичи, такие как
- [[highlighting|Highlighting]]
- [[explain|Explain]]
- [[source-filtering|Source filtering]]
- [[doc-value-fields|Doc value fields]]
- [[versions|Include versions]]
- [[seq-no-primary-term|Sequence numbers and primary terms]]

## Ссылки
https://www.elastic.co/guide/en/elasticsearch/reference/7.8/inner-hits.html