---
share: true
tags: [NET/struct]
---
# Сравнения структур
Если структура используется в сравнениях на равенство, нужно обязательно переопределять ее методы `Equals` и `GetHashCode`, потому что дефолтная реализация для структур использует рефлексию. Resharper напоминает нам это сделать, и сгенерированный им код достаточно хорош.
Для информации см. вторую ссылку
## Ссылки
https://kevingosse.medium.com/performance-best-practices-in-c-b85a47bdd93a
https://devblogs.microsoft.com/premier-developer/performance-implications-of-default-struct-equality-in-c/
