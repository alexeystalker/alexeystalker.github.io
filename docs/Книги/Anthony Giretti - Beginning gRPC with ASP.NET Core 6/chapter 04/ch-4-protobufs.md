---
share: true
tags:
 - gRPC/protobuf
---
# Protobufs
До того, как перейти к реализации gRPC-приложений на ASP.NET Core, необходимо разобраться с языком Protocol Buffers.
Язык Protocol Buffers (или Protobufs) относительно прост. Важно отметить, что сообщения не поддерживают наследование; если вы хотите задать несколько схожих сообщений, нельзя унаследовать их от одного базового.
Файл Protobufs содержит три главных раздела:
- Единичные декларации, такие как версия языка (синтаксиса), опции или импорт другого proto-файла (`.proto` — это расширение файла protobuf);
- декларации сервисов;
- декларации сообщений
#### [[ch-4-individual-declarations|Единичные декларации]]
#### [[ch-4-services-declaration|Декларации сервисов]]
#### [[ch-4-messages-declaration|Декларации сообщений]]
