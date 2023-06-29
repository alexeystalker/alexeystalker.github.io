---
share: true
tags:
 - microservice/pattern
 - microservice/refactoring
---
# Шаблон “Предохранительный слой”
Программный слой, выступающий посредником между двумя доменными моделями, не давая им засорить друг друга своими концепциями.
Предохранительный слой в случае обращения к монолиту в стиле [[request-response-messaging|запрос/ответ]]:
![[Pasted image 20211107185018.png]]
Предохранительный слой в случае подписки на доменные события:
![[Pasted image 20211107185149.png]]

## Ссылки
https://microservices.io/patterns/refactoring/anti-corruption-layer.html