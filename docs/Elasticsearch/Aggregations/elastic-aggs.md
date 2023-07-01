---
tags: [Elasticsearch/aggs]
share: true
---
# Агрегации в Elasticsearch 
Агрегации — это запросы к Elasticsearch, которые как-либо агрегируют значения в документах. Делятся на три вида
1. [[metricaggs|Metric]] — считают "метрики", типа сумм или среднего значения полей
2. [[bucket-aggs|Bucket]] — группируют документы по "корзинам" (buckets or bins) по значениям, диапазонам значений и т.п.
3. [[pipeline-aggs|Pipeline]] — агрегации, которые обрабатывают значения других агрегаций, а не документов

## Ссылки
https://www.elastic.co/guide/en/elasticsearch/reference/7.x/search-aggregations.html
