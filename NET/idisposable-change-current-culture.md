---
share: true
tags: [NET/IDisposable, NET/Culture]
---
# Временно Изменяем CurrentCulture
Иногда для тестирования некоторых функций вам требуется изменить культуру потока, в котором работает приложение. Текущая культура определяется в глобальном свойстве `Thread.CurrentThread.CurrentCulture`.

Создадим класс, который реализует интерфейс IDisposable, чтобы иметь ограниченный блок кода, в котором будет использоваться новая культура:
```csharp
public class TempCulture : IDisposable
{
  CultureInfo _old;

  public TempCulture(CultureInfo culture)
  {
    _old = CultureInfo.CurrentCulture;
    Thread.CurrentThread.CurrentCulture = culture;
  }

  public void Dispose()
  {
    Thread.CurrentThread.CurrentCulture = _old;
  }
}
```
В конструкторе сохраняем текущую культуру в приватном поле. Затем, когда мы вызываем метод `Dispose` (который неявно вызывается при закрытии блока `using`), мы используем это значение для восстановления исходной культуры.

Например, проверим символ валюты:
```csharp
Thread.CurrentThread.CurrentCulture = 
  new CultureInfo("ja-jp");

Console.WriteLine(
  Thread.CurrentThread.CurrentCulture
   .NumberFormat.CurrencySymbol
 );
//￥

using (new TempCulture(new CultureInfo("ru-ru")))
{
  Console.WriteLine(
    Thread.CurrentThread.CurrentCulture
      .NumberFormat.CurrencySymbol
  );
  //₽
}

Console.WriteLine(
  Thread.CurrentThread.CurrentCulture
    .NumberFormat.CurrencySymbol
);
//￥
```
Сначала мы устанавливаем культуру текущего потока на японскую, и получаем символ валюты – йену ￥. Затем временно переходим на русскую культуру и в консоль выводится рубль ₽. Наконец, когда мы выходим за пределы блока `using`, мы снова получаем йену.

## Итого
Использование класса, реализующего `IDisposable`, — хороший способ создать временную среду с характеристиками, отличными от характеристик основной среды.

## Ссылки
https://dev.to/bellonedavide/c-tip-how-to-temporarily-change-the-currentculture-2bp7
