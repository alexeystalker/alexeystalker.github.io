---
tags: [Elasticsearch/query, Elasticsearch/term-level]
share: true
---
# Exists query
Возвращает документы, в которых присутствует указанное поле
```json
GET /_search 
{ 
	"query": { 
		"exists": { 
			"field": "user" 
		} 
	} 
}
```
Считается, что поле "отсутствует" в следующих случаях
- поле равно `null` или `[]`;
- В маппинге для поля указано `"index": false` [[index-map-param|см]];
- длина значения поля больше чем указано в маппинге в параметре `ignore_above` [[ignore-above-map-param|см]];
- Значение сломано (malformed) и в маппинге указан параметр `ignore_malformed` [[ignore-malformed-map-param|см]].
## Ссылки
https://www.elastic.co/guide/en/elasticsearch/reference/7.8/query-dsl-exists-query.html
