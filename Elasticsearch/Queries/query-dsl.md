---
tags: [Elasticsearch/query]
share: true
---
# Query DSL
Головная заметка о Elasticsearch query DSL.
Запрос к Эластику состоит из *выражений (clauses}* 
Выражения могут быть двух типов:
- *Листовые (Leaf)* - выражения, касающиеся значений в полях документа, такие как [[match-query|match]], [[term-query|term]] или [[range-query|range]].
- *Составные (Compound)* - выражения, оборачивающие другое листовое или составное выражение и предназначенные либо для объединения выражений в логическом смысле (такие как [[bool-query|bool]] или [[dis-max-query|dis_max]]), либо для того, чтобы изменить поведение этих выражений (как, например, [[constant-score-query|constant_score]]).

Поведение выражений может различаться в зависимости от того, используются ли они в [[query-filter-context|контексте запроса (query) или фильтра]].

## Ссылки
https://www.elastic.co/guide/en/elasticsearch/reference/7.8/query-dsl.html
