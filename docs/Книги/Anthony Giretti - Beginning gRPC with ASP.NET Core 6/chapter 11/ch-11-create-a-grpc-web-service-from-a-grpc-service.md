---
share: true
tags:
 - gRPC/gRPC-web
---
# Делаем gRPC-web сервис из gRPC сервиса

> [!Note] От меня
> Мой код для этой главы [здесь](https://github.com/alexeystalker/CountryServiceBook/tree/chapter-11)


ASP.NET Core предлагает свою собственную реализацию спецификации gRPC-web. Прежде всего, очень просто сделать приложение gRPC-web из приложения gRPC без изменения поведения приложения, а также без использования прокси. При этом реализация gRPC-web в ASP.NET Core поддерживает [[grpc-client-streaming-call|клиентские потоки]] и [[grpc-duplex-streaming-call|двунаправленные потоки]] в случае, когда для работы приложения не требуется браузер. Таким образмо, требуется сделать другую версию Protobufs для приложений, работающих в браузере (то есть приложений, основанных на HTML/JavaScript). Наконец, gRPC-web поддерживается в Microsoft Azure.
#### [[ch-11-working-with-grpc-web-and-the-net-ecosystem|gRPC-web и экосистема .NET]]
#### [[ch-11-reworking-the-countryservice-grpc-service-for-browser-apps|Переделываем gRPC сервис CountryService для браузерных приложений]]
#### Поддержка ASP.NET Core gRPC-web в Microsoft Azure
Для того, чтобы gRPC-web запустился в Microsoft Azure, нужно включить в Kestrel поддержку HTTP/1 (это можно настроить в appsettings.json). После этого сервис можно размещать в
- Виртуальной машине Windows или Linux;
- Windows App Services или Linux App Services;
- контейнере Windows Docker при помощи Azure Container Instance (ACI)
- кластере Kubernetes.

При этом Windows App Services будут работать даже если вы не включили поддержку HTTP/1, так как там встроен модуль `ASPNetModuleV2`, транслирующий HTTP/2 запросы в HTTP/1. При этом в Linux App Services такого нет. Поэтому, чтобы точно ничего не могло пойти не так, и одновременно работали HTTP/1 и HTTP/2, включите поддержку обоих протоколов (показана только часть файла):
```json
{
  "Kestrel": {
    "EndpointDefaults": {
      "Protocols": "Http1AndHttp2"
    }
  }
}
```