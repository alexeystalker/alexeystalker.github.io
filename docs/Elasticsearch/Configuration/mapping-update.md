---
tags: [Elasticsearch/mapping]
share: true
---
# Меняем mapping в Elastic 
Чтобы обновить маппинг в Эластике, нужно отправить PUT запрос с измененными маппингами, примерно так:
```json
PUT /my-index-000001/_mapping 
{ 
	"properties": { 
		"email": { 
			"type": "keyword" 
		} 
	} 
}
```

