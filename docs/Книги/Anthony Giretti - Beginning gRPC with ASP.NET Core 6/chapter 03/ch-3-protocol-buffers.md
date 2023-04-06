---
share: true
tags:
 - gRPC/protobuf
---
# Protocol Buffers
Клиенту gRPC известны доступные процедуры и их входящие/исходящие параметры благодаря совместному использованию на клиенте и сервере Protocol Buffers - описаний схемы, во многом похожих на WDSL для сервисов SOAP, таких как *WCF (Windows Communication Foundation)*. %% Добавить ссылку на главу 8 %%
Описание Protocol Buffers (Protobuf) хранится в файле `.proto`. Вот пример синтаксиса Protobuf для сервиса `CountryService` с процедурой `GetById()`, получающей параметр `CountrySearchRequest` и возвращающей сообщение `CountryReply`[^1]. 
```protobuf
syntax = "proto3";

service CountryService {
	rpc GetById(CountrySearchRequest) returns (CountryReply) {}
}

message CountrySearchRequest {
	int32 CountryId = 1;
}

message CountryReply {
	int32 Id = 1;
	string Name = 2;
	string Description = 3;
}
```

[^1]: Подробно язык Protobuf разбирается [[ch-4-protobufs|здесь]]