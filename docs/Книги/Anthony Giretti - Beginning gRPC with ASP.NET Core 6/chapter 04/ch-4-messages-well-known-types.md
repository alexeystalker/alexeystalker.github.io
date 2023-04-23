---
share: true
aliases:
 - Well-Known Types
tags:
 - gRPC/protobuf/messages
---
# Известные типы (Well-Known Types)
`Any`, `Value`, `Struct`, `Wrappers`, `Timestamp` и `Duration` — это специальные типы с особым поведением, и поэтому они требуют особого подхода. Google предоставляет специальные файлы для импорта, а затем, при компиляции, используются специальные расширения, известные как *Protobuf’s Well-Known Types*.
## Any
Поля с типом Any предназначены для полей, тип которых неизвестен заранее, но затем требуется *вывести* его. Этот тип предназначен для любых типов, кроме примитивных. C\# не даст упаковать в тип `Any` объект типа, не реализующего `IMessage`. Рассмотрим пример, в котором в сообщении `CountryReply` содержится поле, которое может быть типом `Continent`, но объявлено как `Any`.
> [!important] Важно!
> Для того, чтобы вывести тип, у вас должно быть его определение, так как для вывода требуется, чтобы в сгенерированном коде был дескриптор сообщения для этого типа.

Для того, чтобы объявить поле типа `Any`, нужно импортировать файл `"google/protobuf/any.proto"`. При этом не обязательно на самом деле иметь этот файл где-то на машине — этот файл известен компилятору Protobuf как своего рода “глобальная переменная”.
```protobuf
import "google/protobuf/any.proto";
message CountryReply {
	int32 CountryId = 1;
	google.protobuf.Any Whatever = 2;
}
```
При генерации C\# кода из файла `.proto` создаётся поле `FileDescriptor`, в котором содержатся сигнатуры всех сообщений, содержащихся в этом файле. Сигнатура — это закодированные в base64 имя сообщения и его свойства, сигнатуры должны быть уникальны в рамках пространства имён.
Таким образом, когда мы запаковываем объект какого-то типа в `Any`, мы передаём, кроме значений его свойств, и его сигнатуру, что позволяет при десериализации распаковать переменную типа `Any` в правильный тип. Рассмотрим пример.
```csharp
using Google.Protobuf.WellKnownTypes; //обязательный юзинг
namespace Server;
// Запись
var country = new CountryReply();
country.Whatever =  Any.Pack(new Continent
{
	ContinentId = 1,
	ContinentName = "North America"
});
// Чтение
Continent continent;
if(country.Whatever.Is(Continent.Descriptor)) //Убеждаемся, что дескриптор объекта соответствует
{
	continent = country.Whatever.Unpack<Continent>();
}
// Или
country.Whatever.TryUnpack(out continent);
```
## Wrappers
По умолчанию синтаксис Protobuf не допускает нуллабельных (Nullable) полей. Если вы не запишете значение в поле скалярного типа, при десериализации будет подставлено дефолтное значение. Для этого были введены совместимые с нуллабельными типами .NET *Well-known Type Wrappers* (обёртки, врапперы). Приведём таблицу соответствия типов C\# и врапперов

|Тип C\#|Well-known Type Wrapper|
|---|---|
|`bool?`|`google.protobuf.BoolValue`|
|`double?`|`google.protobuf.DoubleValue`|
|`float?`|`google.protobuf.FloatValue`|
|`int?` |`google.protobuf.Int32Value`|
|`long?`|`google.protobuf.Int64Value`|
|`uint?`|`google.protobuf.UInt32Value`|
|`ulong?`|`google.protobuf.UInt64Value`|
|`string`|`google.protobuf.StringValue`|
|`ByteString`|`google.protobuf.BytesValue`|

Для того, чтобы использовать врапперы, нужно импортировать файл `"google/protobuf/wrappers.proto"`. Далее см. пример:
```protobuf
import "google/protobuf/wrappers.proto";

message Continent {
	int32 ContinentId = 1;
	string ContinentName = 2;
	google.protobuf.BoolValue IsSeparatedBySea = 3;
}
```
При этом в сгенерированном коде будет `bool? IsSeparatedBySea`.
## Value
При помощи Value можно передавать сообщения с полями, тип которых не был известен заранее, а также поля с динамическими типами и коллекциями, допускающими `null`.
Для этого нужно импортировать файл `"google/protobuf/struct.proto"` и использовать тип `google.protobuf.Value`. Вот пример файла `.proto`:
```protobuf
import "google/protobuf/struct.proto";
message CountryReply {
	google.protobuf.Value CountryId = 1;
	google.protobuf.Value Continent = 2;
}
```
В сгенерированном коде тип будет обозначен как `global::Google.Protobuf.WellknownTypes.Value`. Реализует интерфейсы `IMessage<T>`, `IMessage`, `IEquatable<T>`, `IDeepCloneable<T>` и `IBufferMessage`.
Записывать значения в `Value` можно при помощи статических методов, представленных в таблице:

|Тип|Метод|Комментарий|
|---|---|---|
|`Number`|`Value.ForNumber`|Для всех числовых типов .NET|
|`String`|`Value.ForString`||
|`Objects`|`Value.ForStruct`|Не соответствует `struct` из C\#|
|`Boolean`|`Value.ForBool`||
|`Null`|`Value.ForNull`||
|`Collections`|`Value.ForLists`||

`Struct` стоит особняком. Это не `struct` из C\#, а известный в Protobuf тип (*Well-Known Type*), который реализует те же интерфейсы, что и `Value`. Каждое свойство типа `Struct` задаётся в словаре под названием `Fields` типа `MapField<string,Value>()`.
Пример задания определённого выше сообщения `CountryReply`:
```csharp
var country = new CountryReply();
country.CountryId = Value.ForNumber(1);
country.Continent = Value.ForStruct(new Struct
{
	Fields = {
		["ContinentId"] = Value.ForNumber(1),
		["ContinentName"] = Value.ForString("North America"),
		["IsSeparatedBySea"] = Value.ForBool(false)
	}	
});
```
Чтение производится из определённых свойств класса `Value`, содержащих требуемое значение; они приведены в таблице:

|Тип|Свойство|Комментарий|
|---|---|---|
|`Number`|`NumberValue`|Возвращает значение в виде типа .NET `Double`|
|`String`|`StringValue`||
|`Objects`|`StructValue`|Требуется обращаться к свойству `Fields`|
|`Boolean`|`BoolValue`||
|`Null`|`NullValue`||
|`Collections`|`ListValue`||

Приведём пример чтения определённого выше сообщения `CountryReply`:
```csharp
var country = new CountryReply(); //Заполнено из ответа на вызов RPC
var countryModel = new CountryModel
{
	CountryId = Convert.ToInt32(country.CountryId.NumberValue),
	Continent = new ContinentModel
	{
		CountryId = Convert.ToInt32(country.Continent.StructValue.Fields["ContinentId"].NumberValue),
		ContinentName = country.Continent.StructValue.Fields["ContinentName"].StringValue,
		IsSeparatedBySea = country.Continent.StructValue.Fields["IsSeparatedBySea"].BoolValue
	}
};
//Определение моделей
public class CountryModel
{
	public int CountryId { get;set; }
	public ContinentModel Continent { get;set; }
}
public class ContinentModel
{
	public int CountryId { get;set; }
	public string ContinentName { get;set; }
	public bool IsSeparatedBySea { get;set; }
}
```
## Даты и время
Для поддержки типов `DateTimeOffset`, `DateTime` и `TimeSpan` были добавлены такие типы:

|Тип .NET|Protobuf Well-Known Type|
|---|---|
|`DateTimeOffset`|`google.protobuf.Timestamp`|
|`DateTime`|`google.protobuf.TimeStamp`|
|`TimeSpan`|`google.protobuf.Duration`|

Для их использования нужно импортировать `"google/protobuf/timestamp.proto"` и `"google/protobuf/duration.proto"` соответственно.
```protobuf
import "google/protobuf/duration.proto";
import "google/protobuf/timestamp.proto";
message FlightBooking {
	int32 BookingId = 1;
	google.protobuf.Duration FlightDuration = 2;
	google.protobuf.Timestamp DepartureTime = 3;
}
```
В сгенерированном коде `google.protobuf.Timestamp` и `google.protobuf.Duration` превращаются в `Google.Protobuf.WellKnownTypes.Timestamp` и `Google.Protobuf.WellKnownTypes.Duration`, соответственно.
Пример записи и чтения:
```csharp
//запись
var flightBooking = new FlightBooking();
flightBooking.BookingId = 1;
flightBooking.FlightDuration = Duration.FromTimeSpan(new TimeSpan(2, 0, 0)); //2 часа
flightBooking.DepartureTime = TimeStamp.FromDateTime(
	DateTime.SpecifyKind(new DateTime(2021, 7, 1), DateTimeKind.Utc));
	//или .FromDateTimeOffset()

//чтение
var bookingId = flightBooking.BookingId;
var bookingDuration = flightBooking.FlightDuration.ToTimeSpan();
var bookingDepartureTime = flightBooking.DepartureTime.ToDateTime(); //или ToDateTimeOffset()
```
