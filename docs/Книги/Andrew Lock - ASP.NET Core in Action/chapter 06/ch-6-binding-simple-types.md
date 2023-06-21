---
share: true
tags:
 - NET/ASPNETCore/routing
 - NET/ASPNETCore/binding-model
---
# Связывание простых типов
Рассмотрим простую страницу Razor
```csharp
public class CalculateSquareModel : PageModel
{
	public void OnGet(int number)
	{
		Square = number * number;
	}
	public int Square { get; set; }
}
```
Добавим сегмент `{number}` в директиву `@Page`: `@Page "{number}"` для получения шаблона маршрута `CalculateSquare/{number}`. Когда клиент запрашивает URL-адрес `/CalculateSquare/5`, в процессе маршрутизации получим [[route-value|значение маршрута]] `number=5`. В процессе выполнения обработчика страницы, ожидаемый параметр будет найден среди значений маршрута, и значение будет передано в метод.
Для создания [[binding-model|модели привязки]] фреймворк обходит источники привязки по порядку, пытаясь найти соответствующие пары “имя-значение”. Если подходящей пары не нашлось, создаются типы со значениями по умолчанию (`default(T)`).
Теперь рассмотрим страницу с несколькими параметрами.
```csharp
public class ConvertModel : PageModel
{
	public void OnPost( //В оригинале OnGet, хотя далее упоминается отправка формы
		string currencyIn,
		string currencyOut,
		int qty)
	{...}
}
```
Как будут заполнены параметры в зависимости от того, где они переданы

|URL (значения маршрута)|Тело HTTP (значения формы)|Привязанные значения параметров|
|------|------|------|
|`/GBP/USD`| |`currencyIn=GBP` <br> `currencyOut=USD` <br> `qty=0`|
|`/GBP/USD?currencyIn=CAD`|`QTY=50`|`currencyIn=GBP` <br> `currencyOut=USD` <br> `qty=50`|
|`/GBP/USD?qty=100`|`qty=50`|`currencyIn=GBP` <br> `currencyOut=USD` <br> `qty=50`|
|`/GBP/USD?qty=100`|`currencyIn=CAD&currencyOUT=EUR&qty=50`|`currencyIn=CAD` <br> `currencyOut=EUR` <br> `qty=50`|

Как мы видим, значения формы, если они есть, имеют приоритет даже над значениями маршрута.
Если требуется, связыватель модели автоматически преобразует строку в практически любой примитивный тип, а также всё, что имеет преобразователь типов `TypeConverter`[^1] .

[^1]:Подробнее о `TypeConverter` [см. в документации](https://docs.microsoft.com/en-us/dotnet/standard/base-types/type-conversion#the-typeconverter-class)