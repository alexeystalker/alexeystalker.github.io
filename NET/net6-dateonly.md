---
share: true
tags: [NET/NET6]
---
# Тип DateOnly
`DateOnly` - это новая структура в .NET6, представляющая собой _только_ дату: год, месяц и день. 
Конструктор:  `new DateOnly(2021, 5, 31)`
Также есть свойства 
`DayOfWeek` - день недели.
`DateNumber` - по-видимому, день от начала эпохи, используется для вычисления разницы между двумя датами

Можно сочетать с [[net6-timeonly|TimeOnly]] для получения `DateTime`: `dt.ToDateTime(new TimeOnly(0,0)`

Можно получить из [[datetime|DateTime]]:
`DateOnly.FromDateTime(DateTime.Today)`

Не умеет в конверсию часовых поясов (удобно, если это не нужно или нужно избежать)

Лучше совместим с SQL типом date

Умеет в трансляцию из других [[calendar|календарей]]
```csharp
Calendar hebrewCalendar = new HebrewCalendar();
DateOnly d4 = new DateOnly(5781, 9, 16, hebrewCalendar);                   // 16 сивана 5781 г.
Console.WriteLine(d4.ToString("d MMMM yyyy", CultureInfo.InvariantCulture)); // 27 ма
```
