---
share: true
tags: [NET/Enum,NET/boxing]
---
# Enum.Equals делает boxing
`Enum.Equals` следует избегать, так как он делает boxing переменных. Оператор `==` — не делает.
Что делать, когда нельзя использовать `==`, например в дженериках?
Использовать 
```csharp
EqualityComparer<T>.Default.Equals(enum1, enum2);
```
## Ссылки
https://habr.com/ru/company/pvs-studio/blog/568928/
