---
tags: [Elasticsearch/configuration]
share: true
---
# Настройка числа реплик Elasticsearch
```json
PUT /my_source_index/_settings 
{
  "settings": { 
    "index.number_of_replicas": %num%
	}
}
```

