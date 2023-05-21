---
share: true
tags:
 - gRPC/protobuf
---
# Создаём и компилируем файлы Protobuf
В этом разделе мы рассмотрим как создать и скомпилировать файл Protobuf, описывающий набор сервисов приложения. Создадим сервис под названием `CountryService`, который позволит пользователям создавать, изменять, удалять и искать страны, и загружать файл со списком стран, которые требуется сохранить в БД.
Создадим CRUD-операции, на примере которых рассмотрим все возможные [[ch-3-types-of-grpc-services|типы обращения к серверу gRPC]].
Вот что мы реализуем:
- Создание списка стран при помощи двустороннего стриминга (запрос на создание - подтверждение создания);
- Удаление одного или более объектов при помощи стриминга с клиента;
- Получение всех объектов при помощи стриминга с сервера;
- Получение объекта при помощи унарной операции;
- Изменение объекта при помощи унарной операции.

Страна будет представлена объектом со свойствами:
- `ID`;
- `Name`;
- `Flag` (картинка);
- `Creation date` (создание объекта);
- `Update date`.

Согласно данным требованием можно написать такой `.proto` файл:
```protobuf
syntax = "proto3";
package gRPCDemo.v1;
option csharp_namespace = "CountryService.Web.gRPC";

import "google/protobuf/empty.proto";
import "google/protobuf/timestamp.proto";

service CountryService {
	rpc GetAll(google.protobuf.Empty) returns (stream CountryReply) {}
	rpc Get(CountryIdRequest) returns (CountryReply) {}
	rpc Delete(stream CountryIdRequest) returns (google.protobuf.Empty) {}
	rpc Update(CountryUpdateRequest) returns (google.protobuf.Empty) {}
	rpc Create(stream CountryCreationRequest) returns (stream CountryCreationReply) {}
}
message CountryReply {
	int32 Id = 1;
	string Name = 2;
	string Description = 3;
	bytes Flag = 4;
	google.protobuf.Timestamp CreateDate = 5;
	google.protobuf.Timestamp UpdateDate = 6;
}
message CountryIdRequest {
	int32 Id = 1;
}
message CountryUpdateRequest {
	int32 Id = 1;
	string Description = 2;
	google.protobuf.Timestamp UpdateDate = 3;
}
message CountryCreationRequest {
	string Name = 1;
	string Description = 2;
	bytes Flag = 3;
	google.protobuf.Timestamp CreateDate = 5;
}
message CountryCreationReply {
	int32 Id = 1;
	string Name = 2;
}
```
Добавим файл `country.proto` с вышеуказанным содержимым в папку **Protos**.
Теперь убедимся, что `country.proto` правильно зарегистрирован в **CountryService.Web.csproj**:
```xml
<ItemGroup>
	<Protobuf Include="Protos\country.proto" GrpcServices="Server" />
</ItemGroup>
```
> [!Note] От меня
> У меня почему-то файл не зарегистрировался, поэтому пришлось прописать группу руками. Не забудьте проверить, что файл добавился, иначе C\# код не будет сгенерирован при компиляции!

После этого можно запустить сборку солюшена. После сборки можно зайти в папку **\%SolutionName\%/CountryService.Web/obj/Debug/net6.0/Protos** и посмотреть содержимое файлов **Country.cs** и **CountryGrpc.cs** . Вместе содержимое этих файлов образует *заглушки gRPC (gRPC stubs)*.

Более подробно о том, как можно настроить добавление и генерацию файлов, можно посмотреть [здесь](https://github.com/grpc/grpc/blob/master/src/csharp/BUILD-INTEGRATION.md)
