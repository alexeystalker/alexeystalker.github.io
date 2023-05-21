---
share: true
tags:
 - NET/ASPNETCore/publish
 - IIS
---
# Конфигурирование IIS для ASP.NET Core
Прежде всего нужно скачать и установить [ASP.NET Core Windows Hosting Bundle](https://dotnet.microsoft.com/en-us/download/dotnet/thank-you/runtime-aspnetcore-6.0.6-windows-hosting-bundle-installer)
Он включает в себя:
- *среду выполнения .NET*
- *среду выполнения ASP.NET Core*
- *модуль IIS AspNetCore*

После установки пакета необходимо настроить *пул приложений* в IIS. Для ASP.NET Core нужно создать *Неуправляемый (No Managed Code)* пул. Дадим ему название, например `NetCore`.

Далее нужно добавить в IIS новый сайт. Для этого щёлкаем ПКМ по узлу *Сайты* и выбираем **Add Website**. В диалоговом окне указываем имя сайта и путь к папке с файлами. Необходимо именить пул приложений на созданный нами `NetCore`.

Далее предоставим доступ созданному нами пулу приложений. Для этого щёлкаем ПКМ по папке приложения в проводнике Windows, выбираем **Properties**, затем **Security > Edit > Add**, и в текстовом поле вводим `IIS AppPool\NetCore`, нажимаем **Ok**, закрывая все диалоговые окна.

