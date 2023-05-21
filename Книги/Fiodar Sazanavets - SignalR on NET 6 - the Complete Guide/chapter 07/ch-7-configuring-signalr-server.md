---
share: true
tags:
 - NET/SignalR
---
# Настраиваем сервер SignalR
Сперва надо убедиться, что в файле **Program.cs** проекта SignalRServer есть следующие юзинги:
```csharp
using Microsoft.AspNetCore.Http.Connections;
using System.Text.Json.Serialization;
```
Теперь дополним код **Program.cs**, чтобы осветить каждый из следующих видов настроек
#### [[ch-7-top-level-signalr-configuration|Настройка SignalR верхнего уровня]]
#### [[ch-7-json-message-protocol-settings|Настройки протокола JSON]]
#### [[ch-7-applying-advanced-transport-configuration|Применяем расширенные настройки транспорта]]