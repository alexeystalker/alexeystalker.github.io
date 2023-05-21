---
share: true
tags:
 - gRPC/protobuf/messages
---
# Bytes
Язык Protobuf и gRPC поддерживают транспорт бинарных данных. Самый очевидный пример — загрузка файла. Рассмотрим сообщение `CountryImageUpload`, с полями `FileName`, `MimeType` и содержимым файла. 
> [!Note] Примечание
> Передача `MimeType` всегда помогает в таких случаях, особенно если ваше приложение gRPC будет передавать файл в приложение ASP.NET Core, которое, в свою очередь, будет передавать его в клиентский браузер.
```protobuf
message CountryImageUpload {
	string FileName = 1;
	string MimeType = 2;
	bytes Content = 3;
}
```
В сгенерированном коде первые два свойства будут типа `string`, а третье — `ByteString`. Обратите внимание, что значение по умолчанию у третьего свойства — `ByteString.Empty`.
Пример записи и чтения в `ByteString`:
```csharp
var uploadFile = new CountryImageUpload();
//запись
uploadFile.FileName = "Canada_flag.png";
uploadFile.MimeType = "image/png";
//Из массива байт
uploadFile.Content = ByteString.CopyFrom(File.ReadAllBytes("C:\\countries\\flags\\Canada_flag.png"));
//Асинхронно из потока
uploadFile.Content = await ByteString.FromStreamAsync(
	new FileStream("C:\\countries\\flags\\Canada_flag.png", FileMode.Open));
//синхронно из потокв
uploadFile.Content = ByteString.FromStream(
	new FileStream("C:\\countries\\flags\\Canada_flag.png", FileMode.Open));
//Из Base64
uploadFile.Content = ByteString.FromBase64("..."); //Не стал писать Base64 строку
//---
//чтение
var fileName = uploadFile.FileName;
var mimeType = uploadFile.MimeType;
//прочесть как массив байт
var contentInBytes = uploadFile.Content.ToByteArray();
//прочесть как Base64
var contentInBase64 = uploadFile.Content.ToBase64();
//прочесть как поток
var contentInStream = new MemoryStream();
uploadFile.Content.CopyTo(contentInStream);
```
