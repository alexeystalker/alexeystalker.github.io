---
share: true
tags:
 - gRPC/gRPC-web
 - JavaScript
---
# JavaScript библиотеки для gRPC-web
В Improbable и Google были разработаны свои собственные Javascript библиотеки для gRPC-web, и хотя в целом они схожи, существует два основных различия, из-за которых одна может быть предпочтительнее другой.
Первое отличие в том, что библиотека от Improbable, в отличие от библиотеки Google, поддерживает и `XmlHttpRequest` (XHR), и *Fetch API* для вызовов AJAX[^1].
Второе отличие в том, что только библиотека Improbable сама решает (основываясь на возможностях браузера), какой тип транспорта (XHR или Fetch) и какой режим gRPC-web (бинарный или base64). Это удобно, так как передача бинарных данных не поддерживается для [[grpc-server-streaming-call|серверных потоков]].
На основании вышесказанного, в дальнейшем в книге будет использоваться библиотека от Improbable. Если вам интересна библиотека от Google, вы можете прочесть о ней [здесь](https://github.com/grpc/grpc-web). Библиотека от Improbable доступна [здесь](https://github.com/improbable-eng/grpc-web).
С другой стороны, библиотека от Improbable не поддерживает [[ch-3-deadline-and-cancellation|дедлайны]] (но поддерживает отмену) и [[ch-7-interceptors|перехватчики]]. Если вам требуется реализовать перехватчики в JavaScript, вы должны использовать реализацию от Google и следовать [этой инструкции](https://grpc.io/blog/grpc-web-interceptor/) для реализации перехватчиков.
Применение JavaScript библиотеки от Improbable, а также генерация заглушек для JavaScript/TypeScript будет рассмотрена в главе 12.
В заключение рассмотрим сходства и различия двух реализаций в таблице.

|Library|XHR|Fetch|Unary|Server Streaming|Client and Bidirectional Streaming|
|---|---|---|---|---|---|
|Improbable|✔|✔|✔|✔|❌|
|Google text|✔|❌|✔|✔|❌|
|Google binary|✔|❌|✔|❌|❌|

[^1]:[Вот тут](https://www.sitepoint.com/xmlhttprequest-vs-the-fetch-api-whats-best-for-ajax-in-2019/) описано, в чём разница вызовов AJAX при помощи XHR и Fetch API.