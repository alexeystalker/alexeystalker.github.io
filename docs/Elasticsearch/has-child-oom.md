---
tags: [Elasticsearch]
share: true
---
# OutOfMemory в Эластике при has_child запросе
В случае использования джоинов parent-child при фильтре по условию [[has-child-query|has_child]] может очень сильно вырастать потребление памяти в Lucene, и даже привести к крэшу Эластика от OOM.
В логах Эластика стектрейс у ООМ начинается примерно так:
```
"java.lang.OutOfMemoryError: Java heap space",
"at org.apache.lucene.search.join.GlobalOrdinalsWithScoreCollector$Occurrences.increment(GlobalOrdinalsWithScoreCollector.java:363) ~[lucene-join-8.5.1.jar:8.5.1 edb9fc409398f2c3446883f9f80595c884d245d0 - ivera - 2020-04-08 08:56:36]","at org.apache.lucene.search.join.GlobalOrdinalsWithScoreCollector$NoScore$1.collect(GlobalOrdinalsWithScoreCollector.java:258) ~[lucene-join-8.5.1.jar:8.5.1 edb9fc409398f2c3446883f9f80595c884d245d0 - ivera - 2020-04-08 08:56:36]",
"at org.apache.lucene.search.Weight$DefaultBulkScorer.scoreAll(Weight.java:267) ~[lucene-core-8.5.1.jar:8.5.1 edb9fc409398f2c3446883f9f80595c884d245d0 - ivera - 2020-04-08 08:55:42]",
...
```
По ключевым словам `OutOfMemory` и `GlobalOrdinalsWithScoreCollector` ищется  issue, в комментариях к которому описан [workaround](https://github.com/elastic/elasticsearch/issues/64808#issuecomment-730846658):
## Workaround
При построении запроса нужно обязательно указать параметр `min_children`. Даже если условие `min_children` не нужно в запросе, нужно указать `min_children:0`.
## Какие версии затронуты
Баг поправлен в Lucene 8.8 и, cоответственно, в ElasticSearch 7.12. В issue указана версия Elasticsearch 7.4. Следовательно, аффектятся версии как минимум 7.4-7.11
## Ссылки
https://github.com/elastic/elasticsearch/issues/64808
