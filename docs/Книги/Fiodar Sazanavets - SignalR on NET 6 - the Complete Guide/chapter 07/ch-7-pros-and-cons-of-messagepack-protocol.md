---
share: true
tags:
 - NET/SignalR
 - MessagePack
 - Java
 - JavaScript
---
# Плюсы и минусы протокола MessagePack
MessagePack[^1] - это бинарный протокол сериализации данных. Строго говоря, он может использоваться не только в связке с SignalR, но и сам по себе. Обладает схоже с JSON структурой, но в отличие от JSON сериализованный таким образом объект не будет "человекочитаемым". С другой стороны, сериализованные объекты, в силу бинарной природы протокола, будут меньше, следовательно, будут передаваться быстрее.
## Включаем MessagePack на сервере
MessagePack поставляется в виде отдельного NuGet пакета, поэтому прежде всего его нужно установить:
```bash
dotnet add package Microsoft.AspNetCore.SignalR.Protocols.MessagePack
```
Затем, в файле **Program.cs** пишем юзинг:
```csharp
using MessagePack; //Юзинг
```
Далее либо заменим вызов `AddJsonProtocol`, либо добавим после него следующее[^2]:
```csharp
.AddMessagePackProtocol(options => {
	options.SerializerOptions = MessagePackSerializerOptions.Standard
		.WithSecurity(MessagePackSecurity.UntrustedData)
		.WithCompression(MessagePackCompression.Lz4Block)
		.WithAllowAssemblyVersionMismatch(true)
		.WithOldSpec()
		.WithOmitAssemblyVersion(true);
});
```
Этот код демонстрирует некоторые (не все) настройки, которые можно изменять при инициализации[^3]. 
- `SerializerOptions` - объект настроек типа `MessagePackSerializerOptions`;
- `WithSecurity` - так как нет никакого способа проверить валидность данных до начала десериализации, это может быть возможным вектором атаки. Задавая свойства объекта класса `MessagePackSecurity` можно тонко настраивать поведение десериализатора с точки зрения безопасности. Также есть два предустановленных объекта - `TrustedData` для абсолютно доверенных данных и `UntrustedData` для сомнительных;
- `WithCompression` - сериализованные данные можно сжать алгоритмом LZ4 для большей компактности. Возможные варианты: `None`, `Lz4Block` и `Lz4BlockArray`. Последний вариант является предпочтительным. Подробнее разница между вариантами освещена в документации;
- `WithAllowAssemblyVersionMismatch` - можно отключить проверку совпадения версий сборки на клиенте (если версия указана в сообщении) и сервере, или включить, если требуется точное совпадение;
- `WithOldSpec` - позволяет использовать старые спецификации (?);
- `WithOmitAssemblyVersion` - отключает указание версии сборки в сообщении.

## Включаем MessagePack в JavaScript клиенте
На момент написания книги библиотека MessagePack была доступна только при инсталляции через `npm`:
```bash
npm install @microsoft/signalr-protocol-msgpack
```
после установки копируем из папки `node_modules/@microsoft/signalr-protocol-msgpack/dist/browser` файл `signalr-protocol-msgpack.js` или `signalr-protocol-msgpack.min.js` в папку **SignalRServer/wwwroot/lib/signalr** (если такой папки нет - её надо создать). Пусть мы выбрали `signalr-protocol-msgpack.min.js`. Добавляем его в файл **\_Layout.cshtml** следующей строчкой после подключения библиотеки SignalR:
```html
<script src="~/lib/signalr/signalr-protocol-msgpack.min.js"></script>
```
И наконец, в файле **site.js**, в месте, где создаётся объект `connection`, перед вызовом метода `build()` вставляем
```js
.withHubProtocol(new signalR.protocols.msgpack.MessagePackHubProtocol())
```
## Включаем MessagePack в .NET клиенте
добавляем юзинг
```csharp
using MessagePack
```
и добавляем в блок инициализации `hubConnection` перед вызовом метода `Build()` следующее:
```csharp
.AddMessagePackProtocol(options => 
{
	options.SerializerOptions = MessagePackSerializerOptions.Standard
		.WithSecurity(MessagePackSecurity.UntrustedData)
		.WithCompression(MessagePackCompression.Lz4Block)
		.WithAllowAssemblyVersionMismatch(true)
		.WithOldSpec()
		.WithOmitAssemblyVersion(true);
})
```
Как мы видим, настройки идентичны настройкам на сервере.
## Включаем MessagePack в Java клиенте
Пакеты SignalR MessagePack для Java могут быть найдены в репозитории Maven [здесь](https://mvnrepository.com/artifact/com.microsoft.signalr.messagepack/signalr-messagepack). Выбираем нужную нам версию пакета и следуем инструкции для используемой нами системы сборки.
После этого добавляем метод `withHubProtocol()` с указанием объекта протокола MessagePack, вот таким образом:
```java
HubConnection hubConnection = HubConnectionBuilder.create(input)
	.withHubProtocol(new MessagePackHubProtocol())
	.build();
```

## Недостатки протокола MessagePack
Первым недостатком протокола будет то, что он недоступен "из коробки". Для его использования необходимо устанавливать дополнительные пакеты.
Другим недостатком будет то, что этот протокол является регистрозависимым, в отличие от JSON. Это может стать значительной проблемой в случае, когда клиент и сервер написаны на разных языках: в разных языках существуют разные соглашения относительно регистров именования переменных. В .NET это чаще всего PascalCase, а, к примеру, в javaScript это camelCase. Обойти это можно, используя атрибуты для присвоения camelCase-имён свойствам C#-объектов. Подробнее о том, как работает маппинг полей см. в документации[^3].

[^1]: [Сайт проекта MessagePack](https://msgpack.org/) 
[^2]: Напомню, что похожим образом можно [[signalr-newtonsoft-json|подключить  NewtonsoftJson]] в качестве протокола сериализации.
[^3]: [Репозиторий и документация MessagePack-CSharp](https://github.com/neuecc/MessagePack-CSharp)