---
tags: [Elasticsearch/query, Elasticsearch/term-level]
share: true
---
# Term-level queries
*Запросы уровня терма* - запросы к точным значениям полей. Не используют анализ термов (в отличие от [[full-text-queries|полнотекстовых запросов]]).
- [[exists-query|exists]] - возвращает документы, в поле которых есть любое проиндексированное значение
- [[fuzzy-query|fuzzy]]
- [[ids-query|ids]]
- [[prefix-query|prefix]]
- [[range-query|range]]
- [[regexp-query|regexp]]
- [[term-query|term]] - возвращает документы, у которых в указанном поле есть точный терм
- [[terms-query|terms]]
- [[terms-set-query|terms_set]]
- [[type-query|type]]
- [[wildcard-query|wildcard]]