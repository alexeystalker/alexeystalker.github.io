---
share: true
tags: [NET/NET6]
---
# Тип TimeOnly
Тип `TimeOnly` — это структура, которая хранит _только_ время.
Примерно так:
```csharp
TimeOnly t1 = new TimeOnly(16, 30);
Console.WriteLine(t1.Hour);      // 16
Console.WriteLine(t1.Minute);    // 30
Console.WriteLine(t1.Second);    // 0
```
Можно прибавлять часы, минуты или [[timespan|TimeSpan]] (с отрицательным знаком для вычитания).
Часы циклические, то есть при переходе через сутки час станет _меньше_. Однако, есть способ узнать, сколько суток прошло при сложении:
```csharp
TimeOnly t3 = t2.AddMinutes(5000, out int wrappedDays);
```
Можно преобразовывать в TimeSpan (будет отрезок от начала суток до данного).
Можно скомбинировать с [[net6-dateonly|DateOnly]] для получения [[datetime|DateTime]]
Есть метод `IsBetween`, можно узнать, лежит ли данное время между другими двумя.
