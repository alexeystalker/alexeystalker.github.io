---
share: true
tags:
 - gRPC/protobuf/services
---
# Декларация сервисов
Декларация сервисов - это вторая часть файла Protobuf. Здесь мы задаём имя нашего сервиса, а также объявляем процедуры и их параметры. Имя сервиса предваряется ключевым словом `service`, объявления процедур обрамляются фигурными скобками. Каждая функция предваряется ключевым словом `rpc`, далее идёт сигнатура входящего сообщения (параметров) в скобках, затем ключевое слово `returns` и сигнатура исходящего сообщения в скобках, примерно так:
```protobuf
service ServiceName {
	rpc FunctionName1 (InputMessage1) returns (OutputMessage1) {}
	rpc FunctionName2 (InputMessage2) returns (OutputMessage2) {}
}
```
В предыдщей главе мы разбирали [[ch-3-types-of-grpc-services|4 типа сервисов]]. Вот как задаются эти типы в файле `.proto`:
```protobuf
service ServiceName {
	rpc UnaryFunction (InputMessage1) returns (OutputMessage1) {}
	rpc ClientStreamingFunction (stream InputMessage2) returns (OutputMessage2) {}
	rpc ServerStreamingFunction (InputMessage3) returns (stream OutputMessage3) {}
	rpc BidirectionalStreamingFunction (stream InputMessage4) returns (stream OutputMessage4) {}
}
```
