---
share: true
tags: [NET/library, testing]
---
# Генерация случайных данных
Библиотека для генерации случайных данных для тестирования Faker.Net.

Пакет .NET изначально был портом библиотеки Ruby от 2009 года, но во многом разошлась с ней. Библиотека Ruby сама по себе была портом библиотеки Perl Data::Faker от 2007 года. .NET библиотека продолжает использоваться и постоянно обновляется. Причина её популярности, вероятно, связана со множеством вариантов использования и очень простой структурой.

Сначала нужно установить пакет *Faker.Net*. Мы можем сделать это через диспетчер пакетов в Visual Studio или через интерфейс CLI:
```cli
dotnet add package Faker.Net
```

Теперь сгенерируем данные. Допустим, нам нужно создать несколько экземпляров этой модели:
```csharp
class UserProfile
{
  string Name { get; set; }
  int Followers { get; set; }
  string Address { get; set; }
  string Bio { get; set; }
}
```

Здесь нам нужно сгенерировать имя, какое-то случайное число из нужного диапазона, адрес и некоторый случайный текст для биографии.
Мы можем сгенерировать имя из комбинации предопределённых имени, фамилии и отчества, либо сразу полное имя с обращением:
```csharp
var user = new UserProfile();
user.Name = Faker.Name.FullName(NameFormats.WithPrefix);
// "Mrs. Jerod Nader"
```
Для генерации случайных чисел *Faker.Net* использует класс `RandomNumberGenerator` из `System.Security.Cryptography`:
```csharp
user.Followers = Faker.RandomNumber.Next(0, 10000);
// 3452
```
Далее создадим адрес:
```csharp
user.Address = $"{Faker.Address.Country()}, {Faker.Address.City()}";
// "Fiji, Wavaland"
```
Заметьте, что страна и город создаются независимо, поэтому они не пройдут валидацию, если она у вас есть. Также можно создавать индексы, названия улиц и т.п.

Последний шаг – текст для биографии. Можно просто взять несколько предложений из «рыбы» *"Lorem Ipsum…"*:
```csharp
user.Bio = String.Join(" ", Faker.Lorem.Sentences(3));
/* 
"Ea voluptas maiores nihil quia et eum. Vel et eos est architecto rerum est. 
Eum esse voluptatem ab necessitatibus."
*/
```
Кроме того, можно создавать:
- доменные имена:
  ```csharp
  user.CompanyDomain = Faker.Internet.DomainName();
  // "torp.com"
  ```
- email адреса (заметьте, что использовано имя пользователя):
  ```csharp
  user.Email = Faker.Internet.Email(user.Name);
  // "mrs_jerod.nader@hermanborer.name"
  ```
- URL адреса:
  ```csharp
  user.resources = Enumerable.Range(0,2).Select(_ => Faker.Internet.SecureUrl());
  // "https://www.quigleyrobel.us/films/page.aspx"
  // "https://www.braunborer.uk/guide/page.jsp"
  ```
- и многое другое.
## Ссылки
https://blog.elmah.io/easy-generation-of-fake-dummy-data-in-c-with-faker-net/

