---
share: true
tags:
 - NET/ASPNETCore
---
# Ваше первое приложение
Код приложения для этой главы можно найти [здесь](https://github.com/andrewlock/asp-dot-net-core-in-action-2e/tree/master/Chapter02).

---
#### Краткий обзор приложения ASP.NET Core
![[Pasted image 20211202201810.png]]
Приложение ASP.NET Core состоит из: веб-сервера (обычно это Kestrel), [[pipeline|конвейера]] *промежуточного ПО (middleware)* и [[endpoint|компонента конечной точки (endpoint)]].
Сперва запрос извне попадает к веб-серверу, который использует его для создания объекта [[httpcontext|HttpContext]]. Далее веб-сервер передает объект `HttpContext` конвееру промежуточного ПО. Это серия компонентов, предназначенных для выполнения распространенных операций, таких как логирование, обработка исключений или обслуживание статических файлов.
В конце конвейера находится компонент конечной точки. Он отвечает за вызов кода, генерирующего окончательный ответ. В большинстве случаев это будет MVC или Razor Pages.
#### [[ch-2-creating-your-first-app|Создание вашего первого приложения ASP.NET Core]] 
#### [[ch-2-running-web-app|Запуск веб-приложения]]
#### [[ch-2-project-layout|Разбираемся с макетом проекта]]
#### [[ch-2-csproj-file-and-dependencies|Файл проекта .csproj: определение зависимостей]]
#### [[ch-2-program-cs|Класс Program: сборка веб-хоста]]
#### [[ch-2-startup-cs|Класс Startup: настройка вашего приложения]]
#### [[ch-2-razor-pages-response|Создание ответов при помощи Razor Pages]]

