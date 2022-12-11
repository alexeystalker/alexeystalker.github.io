---
share: true
tags:
 - NET/SignalR
---
# Подготовим проект
#### Установим NET 6 и редактор кода
Прежде всего нам понадобится NET 6 SDK. Скачать его можно [здесь](https://dotnet.microsoft.com/en-us/download/dotnet/6.0).
Также нам понадобится редактор кода. Можно выбрать любой редактор с поддержкой синтаксиса C#. Однако, можно использовать и IDE:
- *Visual Studio 2022* - подходит для пользователей Windows 10 и 11, а также macOS, скачать можно [здесь](https://visualstudio.microsoft.com/ru/downloads/);
- *Visual Studio Code* - подходит для Windows, macOS и Linux. Скачать можно на той же странице, что и VS2022;
- *JetBrains Rider* - подходит для Windows, Linux и macOS. Скачать 30-дневный триал можно [здесь](https://www.jetbrains.com/rider/)
#### Добавим HTTPS сертификат для разработчиков
Если необходимо разрабатывать приложение с доступом по HTTPS, можно установить в систему сертификат разработчика: [[ch-18-using-https-development-certificates|см. в книжке Эндрю Лока]]
#### Настраиваем решение (solution)
Создаем папку в нашей файловой системе, например LearningSignalR. Затем, находясь в этой папке, вводим в консоли
```bash
dotnet new sln
```
Далее, создадим проект в папке с решением:
```bash
dotnet new mvc -o SignalRServer
```
Эта команда создаст папку со всеми файлами по умолчанию в ней.
Далее добавим созданный проект к нашему решению:
```bash
dotnet sln add SignalRServer/SignalRServer.csproj
```
#### [[ch-2-setting-up-signalr-hub|Настраиваем хаб SignalR]]
#### [[ch-2-making-hub-strongly-typed|Делаем хаб типизированным]]

## Ссылки
[Официальная документация](https://learn.microsoft.com/en-us/aspnet/core/signalr/hubs?view=aspnetcore-6.0)