---
share: true
tags: [Elasticsearch/query, Elasticsearch/full-text]
---
# Match query
Возвращает документы, содержащие текст, соответствующий (match) запрошенному. Запрошенный текст анализируется перед поиском.

Запрос `match` является стандартным запросом для полнотекстового поиска, включая нечёткий (fuzzy) поиск.
Пример:
```json
GET /_search
{
  "query": {
    "match": {
      "message": {
        "query": "this is a test"
      }
    }
  }
}
```
Параметр: `field` — имя поля, в котором осуществляется поиск (в примере `message`). Является объектом, содержащим параметры.

Некоторые параметры для `field`:
- `query` (обязательный) — текст для поиска. Текст анализируется анализатором запроса (см. ниже);
- `analyzer` (необязательный) Анализатор, используемый для запроса. По умолчанию — анализатор, заданный в [[mapping-params|маппинге]] для поля.

(все параметры см. в документации по ссылке)
## Ссылки
https://www.elastic.co/guide/en/elasticsearch/reference/7.8/query-dsl-match-query.html