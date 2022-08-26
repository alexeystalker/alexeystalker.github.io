---
share: true
tags: [NET/websocket]
---
# Получение сообщения через websocket до конца
Если мы посмотрим на метод получения сообщения `ReceiveAsync()` у класса `Websocket`, то сигнатура у него такая:
```csharp
WebSocketReceiveResult ReceiveAsync(ArraySegment<Byte> buffer, CancellationToken token)
```
(или перегузка с `Memory<byte>`)
Мы выделяем некий буфер и получаем сообщение, записанное внутри буфера:
```csharp
var buffer = new ArraySegment<byte>(new byte[1024 * 4]);
var receiveResult = await client.ReceiveAsync(buffer, cts.Token);
```
Но что делать, если сообщение длиннее буфера? Для этого в `receiveResult` есть поле `EndOfMessage`, которое показывает, всё ли сообщение попало в буфер. Следовательно, чтобы получить всё сообщение, нужно вызвать метод `ReceiveAsync` несколько раз, собрать всё в один массив байт и только после этого прочитать:
```csharp
using var ms = new MemoryStream();
var buffer = new ArraySegment<byte>(new byte[1024 * 4]);
WebSocketReceiveResult receiveResult;
do
{
	receiveResult = await client.ReceiveAsync(buffer, cts.Token);
	ms.Write(buffer.Array, buffer.Offset, receiveResult.Count);
} while(!receiveResult.EndOfMessage);

ms.Seek(0, SeekOrigin.Begin);
using var reader = new StreamReader(ms, Encoding.UTF8);
var messageStr = reader.ReadToEnd();
Console.WriteLine($"Message: {messageStr}");
```
В примере мы передавали текстовое сообщение, поэтому декодируем байты в кодировке UTF8. Если передаем мы некие бинарные данные, то вместо использования `StreamReader`нужно обработать поток соответствующим образом.

## Ссылки
https://docs.microsoft.com/dotnet/api/system.net.websockets.websocket.receiveasync?view=net-6.0
