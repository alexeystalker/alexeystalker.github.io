---
share: true
tags: [NET/struct,NET/boxing,NET/generics]
---
# Структуры и интерфейсы
Рассмотрим код:
```csharp
public class IntValue : IValue
{
}

public void DoStuff()
{
    var value = new IntValue();

    LogValue(value);
    SendValue(value);
}

public void SendValue(IValue value)
{
    // ...
}

public void LogValue(IValue value)
{
    // ...
}
```
Кажется, что было бы разумным сделать IntValue структурой, чтобы избежать ненужных выделений кучи. Однако, так как методы `SendValue` и `LogValue` ожидают реализацию интерфейса (!), при каждой передаче в метод значение value будет бокситься (boxing) при каждом вызове, что приведет к ухудшению ситуации даже в сравнении с использованием класса.

При этом использование шаблонов не вызывает боксинга, таким образом, можно избежать боксинга таким способом:
```csharp
public struct IntValue : IValue
{
}

public void DoStuff()
{
    var value = new IntValue();

    LogValue(value);
    SendValue(value);
}

public void SendValue<T>(T value) where T : IValue
{
    // ...
}

public void LogValue<T>(T value) where T : IValue
{
    // ...
}
```

## Ссылки
https://kevingosse.medium.com/performance-best-practices-in-c-b85a47bdd93a
