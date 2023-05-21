---
tags: [Elasticsearch]
share: true
---
# Clear Cache Api
Используем для очистки кэшей эластика (нужно, например, для профилирования).
Желательно сбрасывать ПЕРЕД КАЖДЫМ замеряемым запросом.
```bash
curl -X POST "localhost:9200/my-index-000001/_cache/clear?pretty"
```
## Варианты запроса
- `POST /<index>/_cache/clear`
- `POST /_cache/clear`  

По умолчанию очищает все три кэша. Однако, можно очищать и по отдельности:
- очищает кэш полей. Можно указать отдельные поля через параметр `fields`
	```bash
	curl -X POST "localhost:9200/my-index-000001/_cache/clear?fielddata=true&pretty"
	```
- очищает кэш поисков (query)
	```bash
	curl -X POST "localhost:9200/my-index-000001/_cache/clear?query=true&pretty"
	```
- очищает кэш запросов (request) 
	```bash
	curl -X POST "localhost:9200/my-index-000001/_cache/clear?request=true&pretty"
	```

Дополнительная инфа в справке по ссылке.
## Ссылки
https://www.elastic.co/guide/en/elasticsearch/reference/7.8/indices-clearcache.html