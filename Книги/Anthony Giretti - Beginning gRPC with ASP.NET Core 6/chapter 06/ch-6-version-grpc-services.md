---
share: true
tags:
 - versioning
 - gRPC
---
# Версионируем сервисы gRPC
В [[ch-4-individual-declarations|Главе 4]] мы упоминали директиву `package`. При помощи этой директивы можно управлять версиями API. 
Создадим две версии файлов нашего [[ch-5-create-and-compile-protobuf-files|сервиса]] `CountryService`. В одной есть управление флагом страны и удаление записи, а в другой — нет, так как мы будем управлять этим через отдельный сервис.
Версия 1:
```protobuf
syntax = "proto3";

package gRPCDemo.v1; //<-- версия 1

option csharp_namespace = "CountryService.Web.gRPC.v1"; //<-- версия 1

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
	google.protobuf.Timestamp CreateDate = 4;
}

message CountryCreationReply {
	int32 Id = 1;
	string Name = 2;
}
```
Версия 2:
```protobuf
syntax = "proto3";

package gRPCDemo.v2; //<-- версия 2

option csharp_namespace = "CountryService.Web.gRPC.v2"; //<-- версия 2

import "google/protobuf/empty.proto";
import "google/protobuf/timestamp.proto";

service CountryService {
	rpc GetAll(google.protobuf.Empty) returns (stream CountryReply) {}
	rpc Get(CountryIdRequest) returns (CountryReply) {}
	rpc Update(CountryUpdateRequest) returns (google.protobuf.Empty) {}
	rpc Create(stream CountryCreationRequest) returns (stream CountryCreationReply) {}
}

message CountryReply {
	int32 Id = 1;
	string Name = 2;
	string Description = 3;
	google.protobuf.Timestamp CreateDate = 4;
	google.protobuf.Timestamp UpdateDate = 5;
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
	google.protobuf.Timestamp CreateDate = 3;
}

message CountryCreationReply {
	int32 Id = 1;
	string Name = 2;
}
```
Засчёт различных версий в директиве `package` и опции `csharp_namespace` можно использовать одинаковые имена сервисов, методов и сообщений. Директива `package` отвечает за различные gRPC URL-адреса, а опция `csharp_namespace` — за различные пространства имён. Предлагается использовать отдельные папки для версий, как показано на картинке:
![[Pasted image 20230408175017.png]]
То же самое проделаем и с реализациями `CountryGrpcService` — разложим их по папкам с версиями:
![[Pasted image 20230408175805.png]]
Наконец, зарегистрируем обе версии в **Program.cs**:
```csharp
using System.IO.Compression;
using Calzolari.Grpc.AspNetCore.Validation;
using CountryService.gRPC.Compression;
using CountryService.Web;
using CountryService.Web.Interceptors;
using CountryService.Web.Validator;
using Grpc.Net.Compression;
using v1 = CountryService.Web.Services.v1; // <--
using v2 = CountryService.Web.Services.v2; // <--

var builder = WebApplication.CreateBuilder(args);

// ...

app.MapGrpcService<v1.CountryGrpcService>(); // <-- 
app.MapGrpcService<v2.CountryGrpcService>(); // <--

// ...
```

И теперь в [[ch-5-using-grpcui|gRPCUi]] доступно обе версии:
![[Pasted image 20230408181829.png]]