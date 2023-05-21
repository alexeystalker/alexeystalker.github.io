---
title: Обзор C# 9
share: true
tags:
 - NET/NET5
---
# Обзор C\# 9
Кратко перечислим самые главные нововведения в C\# 9.
## Init-only свойства
В C\# 9 был представлен новый метод изменения свойства – `init`. Идея в том, чтобы значение свойства можно было задать один раз, при объектной инициализации (object initialization).
## Записи (records)
Новое ключевое слово `record`. Позволяет создать неизменяемый объект ссылочного типа с поведением, как у объекта value-типа. Свойства можно задать как через `init` модификатор, так и через конструктор.
```csharp
public record Product
{
	string Name { get; init; }
	int CategoryId { get; init; }
}
//или
public record Product(string Name, int CategoryId);
```
При этом инициализация записи возможна как через конструктор, так и через объектную инициализацию. Присваивание происходит по значению, с созданием новой копии. При этом существует специальный синтаксис для измения только некоторых свойств записи.
```csharp
var product = new Product {Name = "VideoGame", CategoryId = 1};
var newProduct = product with {CategoryId = 2};
```
Метод `Equals()` у записей переопределен - сравнение происходит по значению полей (как у `struct`).
```csharp
var product = new Product {Name = "VideoGame", CategoryId = 1};
var anotherProduct = new Product {Name = "VideoGame", CategoryId = 1};
product.Equals(anotherProduct); //вернёт true
```
В целом, record полезен при определении *DTO (Data Transfert Object)*, так как обеспечивает неизменяемость и сравнение по значению из коробки.
## Улучшенный pattern matching
Pattern matching улучшился настолько, что позволяет использовать операции сравнения, а также булевы операторы `and`, `or` и `not`, а также комбинировать их.
```csharp
private static int GetTax(Product p) => p.CategoryId switch
{
	0 or 1 => 0,
	> 1 and < 5 => 5,
	> 20 => 15,
	_ => 10
}
```
Оператор `not` также можно использовать в выражении `if` (а также в тернарном выражении)
```csharp
//if
if(p is not ElectronicProduct)
	return 25;

//тернарное выражение
private static int GetDiscountTernary (Product p) => p is not ElectronicProduct ? 25 : 0;
```
## Улучшенное выведение типа
Теперь, можно не указывать тип в момент создания экземпляра (то есть справа от слова `new`), если тип можно вывести. Например, он указан слева от имени переменной (не используется `var`). То же применимо и к условному оператору. Например
```csharp
//Пусть HeadSet и Book унаследованы от Product.
Book aBook = new ("gRPC", 1);
HeadSet headset = new ("Logitech", 2);
Product anotherProduct = aBook ?? headset;
```
## Ковариантный return
Одной из самых недооценённых фич C\# 9 является ковариантный `return`. Обычно в C\#, когда вы унаследуетесь от класса, вы можете переопределить (override) метод, если он объявлен абстрактным или виртуальным, но вы не можете сменить тип возвращаемого значения. Начиная с C\# 9, вы можете вернуть ковариантный тип от изначального типа.
В примере тип `Book`, унаследованный от `Product`, в котором есть абстрактный метод `Order`, возвращающий `ProductOrder`, переопределяет метод `Order` так, что он начинает возвращать тип `BookOrder`, унаследованный от `ProductOrder`.
```csharp
public abstract class Product
{
	protected string Name { get; }
	protected int Id { get; }

	protected Product(string name, int id)
	{
		Name = name;
		Id = id;
	}
	public abstract ProductOrder Order(int quantity);
}
public class Book : Product
{
	public string ISBN { get; }
	public Book(string name, int id, string isbn) : base(name, id)
	{
		ISBN = isbn;
	}
	//Тут переопределяем Order
	public override BookOrder Order(int quantity) => new BookOrder {Quantity = quantity, Product = this};
}
public class ProductOrder
{
	public int Quantity { get; set; }
}
public class BookOrder : ProductOrder
{
	public Book Product { get; set; }
}
```
## Статические лямбды
В C\# 9 появилась возможность делать лямбды статическими. Это позволяет уменьшить аллокации. В следующем примере переменная `_text` захватывается лямбдой, что приводит к дополнительным аллокациям памяти.
```csharp
class Program
{
	private string _text = "{0} is a beautiful product!";
	static void Main()
	{
		PromoteProduct(product => string.Format(this._text, "Surface book 3"));
	}
	private void PromoteProduct(Func<string, string> func)
	{
		Console.WriteLine(func(country));
	}
}
```
В C\#9 можно исправить это, сделав `_text` константой, а лямбду пометить ключевым словом `static`.
```csharp
class Program
{
	private const string _text = "{0} is a beautiful product!";
	static void Main()
	{
		PromoteProduct(static product => string.Format(this._text, "Surface book 3"));
	}
	private void PromoteProduct(Func<string, string> func)
	{
		Console.WriteLine(func(country));
	}
}
```
## Программы верхнего уровня
Программа верхнего уровня (top-level program) - фича C\# 9, позволяющая написать более лёгковесный код в файле **Program.cs**, отбрасывая весь обрамляющий код - определение пространства имён, объявление класса `Program` и метода `Main`.