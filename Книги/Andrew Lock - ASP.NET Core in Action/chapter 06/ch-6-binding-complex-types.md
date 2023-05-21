---
share: true
tags:
 - NET/ASPNETCore/binding-model
---
# Привязка сложных типов
Связыватель моделей может не только преобразовывать строки в [[ch-6-binding-simple-types|примитивные типы]], но и привязывать сложные типы, просматривая свойства, предоставляемые [[binding-model|моделям привязки]]. Это полезно, когда [[page-handler|обработчику страницы]] требуется много параметров, например:
```csharp
public IActionResult OnPost(string firstName, string lastName, string phoneNumber, string email)
```
## Упрощение параметров метода привязкой к сложным объектам
Инкапсулируем отдельные значения в класс:
```csharp
public class UserBindingModel
{
	public string FirstName { get; set; }
	public string LastName { get; set; }
	public string Email { get; set; }
	public string PhoneNumber { get; set; }
}
```
теперь сигнатура обработчика выглядит так:
```csharp
public IActionResult OnPost(UserBindingModel user)
```
Или можно выделить связанное свойство:
```csharp
[BindProperty]
public UserBindingModel User { get; set; }
```
В процессе связывания создается новый экземпляр типа `UserBindingModel`, затем перебираются все свойства этого типа и для каждого свойства ищется пара “имя-значение” в коллекции источников привязки. Также связыватель модели будет искать свойства с префиксом — именем свойства составного типа (в данном случае `User`), то есть `user.FirstName`, `user.LastName` и т.д.
Чтобы привязка состоялась, класс должен иметь публичный конструктор без параметров, а сами свойства должны быть публичными и иметь геттер и сеттер.
## Привязка коллекций и словарей
Также можно настраивать привязку к коллекциям, спискам и словарям.
Например, чтобы передать список, создадим обработчик страницы, принимающий список:
```csharp
public void OnPost(List<string> currencies);
```
Отправлять данные в этот метод можно в нескольких форматах:
- `currencies[index]`, где `currencies` — имя параметра для привязки, а `index` — индекс элемента, например `currencies[0]=GBP&currencies[1]=USD`;
- `[index]`, если выполняется привязка только к одному списку: `[0]=GBP&[1]=USD`;
- `currencies` — можно опустить индекс: `currencies=GBP&currencies=USD`.

В случае словаря ключ заменяет индекс.
## Привязка загрузки файлов с помощью IFormFile
Если необходимо загрузить файл, это можно сделать с использованием интерфейса `IFormFile`:
```csharp
public void OnPost(IFormFile file);
```
Или в случае нескольких файлов:
```csharp
public void OnPost(IEnumerable<IFormFile> file);
```
Внимание! Этот метод не подходит для загрузки больших файлов!
Для загрузки больших файлов лучше использовать потоковую передачу.
