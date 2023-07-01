---
tags: [Elasticsearch/aggs]
share: true
---
# Missing aggregation
“Однокорзинная” агрегация, создает корзину c документами, в которых отсуствует поле (или имеет [[null-config|сконфигуренное]] NULL значение).
```json
{
    "aggs": {
        "no_income": {
            "missing": {"field": "income"}
        }
    }
}
```
Хорошо сочетается с [[range-agg|range]] агрегацией, когда нужно выделить документы без значений в отдельную колонку:
```json
{
    "aggs": {
        "income_ranges": {
            "range": {
                "field": "income",
                "keyed": true,
                "ranges": [
                    { "from": 0, "to": 1 },
                    { "from": 1, "to": 100000 },
                    { "from": 100000, "to": 250000 },
                    { "from": 250000, "to": 500000 },
                    { "from": 500000, "to": 1000000 },
                    { "from": 1000000, "to": 2000000 },
                    { "from": 2000000, "to": 5000000 },
                    { "from": 5000000, "to": 10000000 },
                    { "from": 10000000, "to": 20000000 },
                    { "from": 20000000, "to": 40000000 },
                    { "from": 40000000, "to": 100000000 },
                    { "from": 100000000, "to": 500000000 },
                    { "from": 500000000, "to": 1000000000 },
                    { "from": 1000000000 }
                    ]
                }
        },
        "no_income": {
            "missing": {"field": "income"}
        }
    }
}
```
## Ссылки
https://www.elastic.co/guide/en/elasticsearch/reference/7.8/search-aggregations-bucket-missing-aggregation.html
