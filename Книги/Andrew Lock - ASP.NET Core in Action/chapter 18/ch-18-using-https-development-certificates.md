---
share: true
tags:
 - NET/ASPNETCore/HTTPS
 - HTTPS
---
# Использование HTTPS-сертификатов для разработки
При первом запуске команды `dotnet` с помощью .NET SDK, SDK устанавливает на компьютер HTTPS-сертификат для разработки. Однако, этот сертификат не является *доверенным*.
Чтобы начать *доверять* сертификату, в Windows или macOS достаточно выполнить команду
```bash
dotnet dev-certs https -trust
```

Эта команда регистрирует сертификат в “хранилище сертификатов” операционной системы.
В случае с Linux всё зависит от конкретного дистрибутива операционной системы. Для Docker можно обратиться к [документации от Microsoft](https://docs.microsoft.com/aspnet/core/security/enforcing-ssl?view=aspnetcore-6.0&tabs=visual-studio#how-to-set-up-a-developer-certificate-for-docker).

Если используется Windows, Visual Studio и IIS Express, то предупреждения о недоверенном сертификате может и не появиться, так как Visual Studio доверяет сертификату IIS, а IIS Express, в свою очередь, выступает как обратный прокси-сервер и сам выполняет настройку SSL/TLS.