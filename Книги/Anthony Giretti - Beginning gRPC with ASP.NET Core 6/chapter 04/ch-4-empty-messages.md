---
share: true
tags:
 - gRPC/protobuf/messages
---
# Пустые сообщения
Как ранее уже упоминалось, язык Protobuf требует, чтобы у функции обязательно были указаны входное и выходное сообщение, даже если функции это не требуется. Для этого случая есть две возможности.
- Создать сообщение без свойств;
- Использовать Well-Known Type `Empty`.

Например, пусть у нас есть функция `GetAll`, которая не принимает входных параметров. Определим специальное сообщение `EmptyRequest`.
```protobuf
service CountryService {
	rpc GetAll(EmptyRequest) returns (CountriesReply) {}
}
message EmptyRequest {

}
message CountriesReply {
	repeated CountryReply = 1;
}
message CountryReply {
	int32 CountryId = 1;
	string CountryName = 2;
}
```
При этом для вызова такой функции нужно создать объект `EmptyRequest` и передать его:
```csharp
// ...
var emptyRequest = new EmptyRequest();
var countries = await countryClient.GetAllAsync(emptyRequest);
```
Также есть вторая возможность - использовать `Empty`:
```protobuf
import "google/protobuf/Empty.proto";

service CountryService {
	rpc GetAll(google.protobuf.Empty) returns (CountriesReply) {}
}
message CountriesReply {
	repeated CountryReply = 1;
}
message CountryReply {
	int32 CountryId = 1;
	string CountryName = 2;
}
```
Однако, как и в первом случае, объект пустого сообщения должен быть создан и передан:
```csharp
// ...
var emptyRequest = new Empty();
var countries = await countryClient.GetAllAsync(emptyRequest);
```
