---
tags: [Elasticsearch/configuration]
share: true
---
# Закрыть на запись индекс Elasticsearch

Чтобы закрыть индекс Эластика на запись извне (изменение настроек остаётся), нужно сделать запрос
```json
PUT /my_source_index/_settings 
{
  "settings": { 
    "index.blocks.write": true
	}
}
```
Нужно, к примеру для [[elastic-clone|клонирования]]
## Ссылки
https://www.elastic.co/guide/en/elasticsearch/reference/master/indices-clone-index.html
