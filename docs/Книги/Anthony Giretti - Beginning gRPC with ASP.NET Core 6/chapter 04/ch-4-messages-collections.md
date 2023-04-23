---
share: true
tags:
 - gRPC/protobuf/messages
---
# Коллекции
Коллекции поддерживаются языком Protobuf в виде списков и словарей.
## Списки
Задаются при помощи ключевого слова `repeated`:
```protobuf
message CountrySearchRequest {
	repeated int32 CountryIds = 1;
}
```
В сгенерированном коде поле получает тип (для примера выше) `RepeatedField<int>` из сборки `Google.Protobuf`. Тип `RepeatedField<T>` реализует такие интерфейсы как `IList<T>`, `ICollection<T>`, `IEnumerable<T>`, `IEnumerable`, `IList`, `ICollection`, `IEquatable<RepeatedField<T>>`, `IReadOnlyList<T>` и `IReadOnlyCollection<T>`, а также ещё один интерфейс из `Google.Protobuf` — `IDeepCloneable<RepeatedField<T>>`.
## Словари
Словарь задаётся при помощи конструкции `map<TKey, TValue>`, и в сгенерированном коде превращается в `MapField`. В примере зададим словарь с `int32` в качестве типа ключа и `string` в качестве типа значения:
```protobuf
message CountryReply {
	map<int32, string> countries = 1;
}
```
`MapField<TKey, TValue>` реализует `IDictionary<TKey, TValue>`, а также, как и многие другие, `IEquatable<MapField<TKey, TValue>>` и `IDeepCloneable<TKey, TValue>>`.
