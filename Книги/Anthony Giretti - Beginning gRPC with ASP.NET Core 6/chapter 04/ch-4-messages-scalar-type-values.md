---
share: true
tags:
 - gRPC/protobuf/messages
---
# Скалярные типы
Отметим, что параметры скалаярных типов нельзя задавать напрямую в декларации функции; они должны обязательно быть частью декларации сообщения.
Приведём таблицу соответствия типов Protocol Buffers, типов C\# и их значений по умолчанию.

|Protocol Buffer Type|C\# Type|C\# Default Value|
|---|---|---|
|`double`|`double`|`0`|
|`float`|`float`|`0`|
|`int32`|`int`|`0`|
|`int64`|`long`|`0`|
|`uint32`|`uint`|`0`|
|`uint64`|`ulong`|`0`|
|`sint32`|`int`|`0`|
|`sint64`|`long`|`0`|
|`fixed32`|`uint`|`0`|
|`fixed64`|`ulong`|`0`|
|`sfixed32`|`int`|`0`|
|`sfixed64`|`long`|`0`|
|`bool`|`bool`|`false`|
|`string`|`string`|`string.Empty`|
|`bytes`|`ByteString`|`empty bytes`|