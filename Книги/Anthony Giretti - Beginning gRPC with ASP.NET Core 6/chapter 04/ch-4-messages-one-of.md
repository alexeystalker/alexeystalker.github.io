---
share: true
tags:
 - gRPC/protobuf/messages
---
# One of
`One of` — это возможность указать, что поле в сообщении будет *одного из перечисленных типов*. К примеру, если при построении ответа произошла ошибка, и вы хотите вернуть ошибку вместо заполенного поля. Этот механизм позволяет уменьшить размер сообщения, не отправляя поля, значения которых по тем или иным причинам (например, из-за ошибки) недоступны.
Предположим, у нас объявлены сообщения `Country`, `Continent` и `Error` в соответствующих файлах. Тогда можно собрать такое сообщение:
```protobuf
import "Protos/country.proto";
import "Protos/continent.proto";
import "Protos/error.proto";
message CountryOrContinentReply {
	oneof countryOrContinent {
		Country Country = 1;
		Continent Continent = 2;
		Error Error = 3;
	}
}
```
Интересно, что в этом случае сгенерируется тип `CountryOrContinentReply` со свойствами `Country`, `Continent` и `Error`, а также со специальным `enum` `CountryOrContinentOneofCase` такого вида:
```csharp
public enum CountryOrContinentOneofCase
{
	None = 0,
	Country = 1,
	Continent = 2,
	Error = 3
}
```
и соответствующим свойством `CountryOrContinentCase`. Конкретное значение в этом свойстве будет определять, заполнено соответствующее свойство, или нет. Если значение `None` — везде будут `null`-ы. Если `Country` — `null`-ы будут везде, кроме `Country` и так далее.
Чтобы было совсем понятно, приведём пример сгенерированного кода для `Country` (для остальных полей будет то же самое с поправкой на тип):
```csharp
//я добавил форматирование, чтобы было удобнее читать. В книге форматирования нет
public const int CountryFieldNumber = 1;
[global::System.Diagnostics.DebuggerNonUserCodeAttribute]
public global::Apress.Sample.gRPC.Country Country 
{
	get 
	{ 
		return countryOrContinentCase_ == CountryOrContinentOneofCase.Country 
			? (global::Apress.Sample.gRPC.Country) countryOrContinent_ 
			: null;
	}
	set 
	{
		countryOrContinent_ = value;
		countryOrContinentCase_ = value == null
			? CountryOrContinentOneofCase.None
			: CountryOrContinentOneofCase.Country;
	}
}
```
Видно, что при присвоении значения свойству также выставляется значение `CountryOrContinentOneofCase`, а при чтении значение из backing field достанется только при нужном значении в `CountryOrContinentOneofCase`.
Пример записи/чтения
```csharp
//запись
var countryOrContinentReply = new CountryOrContinentReply();
countryOrContinentReply.Continent = new Continent
{
	ContinentId = 1,
	ContinentName = "Americas"
};
//чтение
switch(countryOrContinentReply.CountryOrContinentCase)
{
	case CountryOrContinentReply.CountryOrContinentOneofCase.Country:
		Console.Write("Country found");
		break;
	case CountryOrContinentReply.CountryOrContinentOneofCase.Continent:
		Console.Write("Continent found");
		break;
	case CountryOrContinentReply.CountryOrContinentOneofCase.Error:
		Console.Write("Error happened");
	default:
		throw new ArgumentException("Unhandled response");
}
```
