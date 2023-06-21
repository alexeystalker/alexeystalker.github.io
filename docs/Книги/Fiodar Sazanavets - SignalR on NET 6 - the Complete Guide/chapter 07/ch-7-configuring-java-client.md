---
share: true
tags:
 - NET/SignalR
 - Java
---
# Настраиваем клиент Java
Для клиента Java логирование сконфигурировано при помощи следующего пакета:
```
org.slf4j:slf4j-jdk14
```
Если этот пакет у вас не установлен, в консоли приложения могут появляться ошибки, не влияющие на работу приложения.
Добавим конфигурирование к клиенту SignalR для Java, который мы создали в [[ch-4-setting-up-java-client|главе 4]]. Для этого откроем файл **App.java** и заменим код создания объекта `hubConnection` следующим:
```java
HubConnection hubConnection = HubConnectionBuilder.create(input)
	.withHeader("Key", "value")
	.shouldSkipNegotiate(true)
	.withHandshakeResponseTimeout(30*1000)
	.withTransport(TransportEnum.WEBSOCKETS)
	.build();
```
Вот какие нам доступны настройки:
- `withHeader` — добавляет заголовок к HTTP-запросу;
- `shouldSkipNegotiate` — если `true` — при использовании транспорта WebSockets не использует предварительный negotiate запрос;
- `withHandshakeResponseTimeout` — таймаут для ожидания ответа сервера при первичном взаимодействии (handshake). При превышении это времени соединение завершается. Значение задаётся в миллисекундах;
- `withTransport` — задаёт транспорты, поддерживаемые клиентом. По умолчанию поддерживаются все три транспорта и WbSocket имеет приоритет.

Также у `hubConnection` существуют  динамически изменяемые свойства, вот они:
```java
hubConnection.setServerTimeout(30000);
hubConnection.withHandshakeResponseTimeout(15000);
hubConnection.setKeepAliveInterval(15000);
```
Здесь:
- `getServerTimeout / setServerTimeout` — получает или задаёт интервал ожидания сообщения от сервера (включая пинги). Если за указанный интервал сообщений нет, соединение закрывается. По умолчанию 30 секунд;
- `withHandshakeResponseTimeout` — время ожидания ответа сервера при первичном взаимодействии (handshake). По умолчанию 15 секунд;
- `getKeepAliveInterval / setKeepAliveInterval` — интервал отправки пингов на сервер. По умолчаню 15 секунд.