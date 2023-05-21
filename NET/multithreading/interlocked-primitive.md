---
tags: [NET/multithreading]
share: true
---
# Interlocked
Статический класс с набором методов, представляющих атомарные операции.
Применять для обеспечения атомарности операции, не являющейся атомарной "обычным способом".
Является наиболее легковесным примитивом синхронизации.
## Методы
Все методы являются атомарными операциями
### Add
```csharp
public static int Add(ref int location1, int value);
```
складывает 2 целых числа и заменяет первое число (`location1`) на сумму. Возвращает сумму. Есть перегрузки для `int`, `long`, `uint`, `ulong`.
### And
```csharp
public static int And(ref int location1, int value);
```
выполняет побитовое "И" над двумя числами и заменяет первое число (`location1`) на результат операции. Возвращает исходное значение `location1`. Есть перегрузки для `int`, `long`, `uint`, `ulong`.
### CompareExchange
```csharp
public static int CompareExchange(ref int location1, int value, int comparand);
```
сравнивает значения `location1` и `comparand`, и, если они равны, заменяет значение `location1` на значение `value`. Возвращается *исходное* значение `location1`, вне зависимости от того, произошел обмен или нет.
Есть перегрузки: `int`, `long`, `uint`, `ulong`, `float`, `double`, `object`(сравнение по ссылке), `IntPtr`, а также generic версия `CompareExchange<T>(ref T,T,T) where T : class` (сравнение по ссылке).
Пример применения:
```csharp
public class ExampleClass
{
	private int commonVar = 0;
	public void ExampleMethod()
	{
		var cVar = commonVar; //Сохраняем значение локально
		var newVal = ...; //Вычисляем новое значение, возможно с временнЫми затратами
		if(cVar != Interlocked.CompareExchange(ref commonVar, newVal, cVar))
		{
			//Пока мы вычисляли новое значение, кто-то успел изменить commonVar, решаем, что делать
			...
		}
	}
}

```
### Decrement
```csharp
public static int Decrement(ref int location);
```
уменьшает значение переменной на 1. Возвращает результат операции. Перегрузки для `int`, `long`, `uint`, `ulong`.
### Exchange
```csharp
public static int Exchange(ref int location1, int value);
```
заменяет значение переменной `location1` на `value`, возвращая оригинальное значение. Перегрузки для `int`, `long`, `uint`, `ulong`, `float`, `double`, `object`, `IntPtr` а также generic версия `T Exchange<T> (ref T location1, T value) where T : class` . 
### Increment
```csharp
public static int Increment(ref int location);
```
увеличивает значение переменной на 1. Возвращает результат операции. Перегрузки для `int`, `long`, `uint`, `ulong`.
### MemoryBarrier
```csharp
public static void MemoryBarrier();
```
синхронизирует доступ к памяти: процессор, выполняющий текущий поток не может менять местами выполнение инструкций таким образом, что операции доступа к памяти до вызова `MemoryBarrier()` выполняются после операций после вызова `MemoryBarrier()`. Обёртка для [[volatile-primitives|Thread.MemoryBarrier()]].
### MemoryBarrierProcessWide
```csharp
public static void MemoryBarrierProcessWide();
```
синхронизирует доступ к памяти для ВСЕХ процессоров: чтение и запись всех процессоров не могут быть переставлены.
В отличие от обычного `MemoryBarrier()` имеет большой оверхед и должен использоваться только там, где это действительно необходимо.
Является обёрткой `FlushProcessWriteBuffers` для Windows и `sys_membarrier` для Linux.
### Or
```csharp
public static int Or (ref int location1, int value);
```
выполняет побитовое "ИЛИ" над двумя значениями и сохраняет результат в `location1`. Возвращает оригинальное значение `location1`. Перегрузки для `int`, `long`, `uint`, `ulong`.
### Read
```csharp
public static long Read (ref long location);
```
загружает 64-битное значение. Нужен для 32-битных систем, для которых 64-разрядная операция чтения не является атомарной.
## Ссылки
https://docs.microsoft.com/ru-ru/dotnet/api/system.threading.interlocked?view=net-6.0