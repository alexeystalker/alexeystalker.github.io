---
share: true
tags:
 - gRPC
---
# Создание клиента gRPC
В этой главе мы разберём моменты, связанные с написанием клиента для gRPC-сервера.

#### Создадим консольное приложение
Тут никаких трудностей нет. Создаём консольное приложение (назовём его `CountryService.Client`), добавляем в него три NuGet-пакета:
- `Google.Protobuf` — классы для работы с Protobuf, например [[ch-4-messages-well-known-types|Well-Known Types]];
- `Grpc.Net.ClientFactory` — фабрика и всё необходимое для создания gRPC-клиентов;
- `Grpc.Tools` — для компиляции файлов Protobuf

#### [[ch-7-compile-protobuf-files-and-generate-grpc-clients|Компилируем файлы Protobuf и генерируем клиенты]]
#### [[ch-7-consume-grpc-services-with-net-6|Используем сервисы gRPC в NET 6]]
#### [[ch-7-optimize-performance|Оптимизируем производительность]]
#### [[ch-7-get-message-validation-errors-from-the-server|Получаем ошибки валидации сообщенй с сервера]]