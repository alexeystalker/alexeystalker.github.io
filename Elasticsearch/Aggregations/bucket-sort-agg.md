---
tags: [Elasticsearch/aggs]
share: true
---
# bucket_sort agg
Сортировка бакетов исходной агрегации. Запрос выглядит примерно так:
```json
{
  "aggs": {
    "okveds": {
      "terms": {
        "field": "okved-all-exact",
        "size": 10000
      },
      "aggs": {
        "truncate": {
          "bucket_sort": {
            "sort": [
              {"_count": {"order": "asc"} }
            ],
            "from": 10,
            "size": 40
          }
        }
      }
    }
  }
}
```
Важно, что `bucket_sort` работает только внутри другой агрегации - в нашем случае внутри [[terms-agg|terms]].
Здесь: `sort` - поля, по которым производится сортировка (для `terms` доступны поля `_key` и `_count` для ключа и значения), `from` - смещение, от которого начинать выбор бакетов, `size` - сколько взять. Замечу, что все поля необязательные, так что `bucket_sort` можно использовать для, например, получения top40 бакетов.
## Ссылки
https://www.elastic.co/guide/en/elasticsearch/reference/7.8/search-aggregations-pipeline-bucket-sort-aggregation.html