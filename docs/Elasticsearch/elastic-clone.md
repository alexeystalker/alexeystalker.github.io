---
tags: [Elasticsearch]
share: true
---
# Как склонировать индекс Elasticsearch
Чтобы склонировать индекс эластика, надо сделать запрос
```http
POST /<index>/_clone/<target-index>
```
Перед клонированием нужно [[blocks-write|закрыть]] индекс на запись. Также надо [[number-of-replicas|выставить]] число реплик равным числу исходного индекса (по умолчанию ставится 1).
## ссылки
https://www.elastic.co/guide/en/elasticsearch/reference/master/indices-clone-index.html
