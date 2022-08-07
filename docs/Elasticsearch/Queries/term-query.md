---
tags: [Elasticsearch/query, Elasticsearch/term-level]
share: true
---
# Term query
Ищет документы с точным вхождением терма. Как правило, это поля типа `keyword`.
```json
{
	"query": {
		"term": {
			"opf-exact": {
				"value": "50102"
			}
		}
	}
}
```
В отличие от многих других запросов, имя поля указывается не как значение параметра `field`, а как имя поля объекта.
## Ссылки
https://www.elastic.co/guide/en/elasticsearch/reference/7.8/query-dsl-term-query.html
