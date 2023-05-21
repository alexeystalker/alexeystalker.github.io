---
share: true
tags:
 - NET/ASPNETCore/publish
 - Linux
 - docker
---
# Запуск приложения ASP.NET Core за обратным прокси-сервером в Linux
Запуск приложения в Linux во многом аналогичен запуску приложения [[ch-16-publishing-to-iis|с помощью IIS]].
1. *Опубликуйте приложение* при помощи [[ch-16-running-vs-publishing|команды]] `dotnet publish`;
2. *Установите необходимые компоненты на сервер*. [Инструкция](https://docs.microsoft.com/en-gb/dotnet/core/install/linux);
3. *Скопируйте приложение на сервер*;
4. *Настройте обратный прокси-сервер*;
5. *Настройте инструмент управления процессами для приложения*. IIS действует и как [[reverse-proxy|обратный прокси-сервер]], и как диспетчер процессов, перезапуская приложение в случае сбоя; в Linux обычно требуется настроить отдельное ПО для этого.

> [!Info]+ Запуск приложений ASP.NET Core в Docker
> ASP.NET Core хорошо подходит для развертывания в контейнерах, однако переход к Docker требует сдвига в методологии развертывания. Вот некоторые ресурсы, с которыми рекомендуется ознакомиться:
> - Книга [Docker in Practice](https://livebook.manning.com/book/docker-in-practice-second-edition/about-this-book/);
> - Docker также можно использовать и в Windows. Бесплатная книга [Введение в контейнеры Windows](https://download.microsoft.com/download/A/1/3/A13A2B9E-D47C-4F93-B180-F7F9CD3382A7/Introduction_to_Containers_ebook.pdf);
> - [Документация](https://docs.microsoft.com/aspnet/core/host-and-deploy/docker/?view=aspnetcore-6.0) от Microsoft;
> - [Серия постов](https://www.stevejgordon.co.uk/docker-dotnet-developers-part-1) от Стива Гордона о Docker для разработчиков ASP.NET Core

Как выполнить пункты 4 и 5? Зависит от того, какие обратные прокси и диспетчеры процессов вы хотите использовать.Вот руководства по настройке [NGINX и systemd](https://docs.microsoft.com/aspnet/core/host-and-deploy/linux-nginx?view=aspnetcore-6.0) и [Apache и systemd](https://docs.microsoft.com/aspnet/core/host-and-deploy/linux-apache?view=aspnetcore-6.0) от Microsoft.