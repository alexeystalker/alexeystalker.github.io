---
title: CORS
share: true
tags:
 - CORS
 - protocol
---
# CORS — Cross-origin resource sharing
*Cross-origin resource sharing (CORS)* — протокол, позволяющий JavaScript делать запросы из одного домена в другой.
> [!tip] Определение
> *Источники* считаются одинаковыми, если у них совпадают протокол (HTTP или HTTPS), домен и порт. Если приложение пытается получить доступ к ресурсу с помощью JavaScript и источники не идентичны, браузер блокирует запрос.

Чтобы запрос выполнился, он должен быть обращён к источнику, идентичному тому, откуда получен выполняющийся код JavaScript, либо в случае, если источник кода JavaScript допущен к выполнению запроса принимающим источником. Правила, по которым определяется, допустим ли источник запроса, задаются при помощи *политик* CORS.

## Ссылки
[Спецификация](https://fetch.spec.whatwg.org/#http-cors-protocol)
[Описание на developer.mozilla.org](https://developer.mozilla.org/ru/docs/Web/HTTP/CORS)
[[ch-18-calling-web-apis-from-other-domains-using-cors|Раздел из книги Э. Лока, посвящённый настройке CORS в ASP.NET Core]]