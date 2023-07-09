---
share: true
tags: [NET/lambda]
---
# Lambda функции предпочтительнее MethodGroup
Рассмотрим вот такой код:
```csharp
public IEnumerable<int> GetItems()
{
	return _list.Where(i => Filter(i));
}

private static bool Filter(int element)
{
	return i % 2 == 0;
}
```
Resharper обычно предлагает заменить лямбду на MethodGroup (то есть на `return _list.Where(Filter);`). Однако, MethodGroup — это синтаксический сахар, за которым скрывается следующее:
```csharp
public IEnumerable<int> GetItems()
{
    return _list.Where(new Predicate<int>(Filter));
}
private static bool Filter(int element)
{
    return i % 2 == 0;
}
```
Как видим, на каждый вызов создаётся объект `Predicate` со всем положенным оверхедом. Для лямбд же применяется специальная оптимизация, которая (для вызова статических методов) создаёт предикат только один раз и кэширует его в статическое поле.
Если метод НЕ статический, можно кэшировать его самому:
```csharp
private Predicate<int> _filter;

public Constructor()
{
    _filter = new Predicate<int>(Filter);
}

public IEnumerable<int> GetItems()
{
    return _list.Where(_filter);
}

private bool Filter(int element)
{
    return i % 2 == 0;
}
```
## Ссылки
https://kevingosse.medium.com/performance-best-practices-in-c-b85a47bdd93a
