---
share: true
aliases:
 - POCO
tags:
 - NET
 - definition
---
# POCO — Plain Old CLR Object
*Простой старинный объект CLR*, чаще POCO — классический акроним для обозначения простого класса-сущности, чаще всего имеющего только свойства с геттерами и сеттерами, и не унаследованный от других классов (за исключением `Object`).
Приведем пример классического POCO, описывающего некого пользователя системы:
```csharp
public class User
{
	public Guid UserId { get; set; }
	public string Username { get; set; }
	public string Email { get; set; }
}
```


## Ссылки
[ru-wiki](https://ru.wikipedia.org/wiki/Plain_old_CLR_object)
[en-wiki](https://en.wikipedia.org/wiki/Plain_old_CLR_object)

