---
share: true
tags:
 - NET/ASPNETCore/Identity
---
# Что такое ASP.NET Core Identity?
ASP.NET Core Identity — система для хранения данных пользователя и управления ими в своей базе данных. ASP.NET Core Identity берет на себя большую часть шаблонного кода, связанного с [[authentication|аутентификацией]].
По умолчанию используется [[ch-12-saving-data-with-ef-core|EF Core]].
Сервисы, предоставляемые Identity:
- Схема БД для хранения пользователей и утверждений;
- Создание пользователя в БД;
- Проверка пароля и и контроль правил для пароля;
- Блокировка учётной записи пользователя (для предотвращения атак методом перебора);
- Генерация кодов двухфакторной аутентификации и управление ими;
- Генерация токенов для сброса пароля;
- Сохранение дополнительных утверждений в БД;
- Управление сторонними поставщиками идентификационной информации.

А вот что требуется реализовать со стороны разработчика:
- Пользовательский интерфейс для выполнения входа, создания и управления пользователями (Razor pages или контроллеры). Включено в дополнительный пакет, предоставляющий UI по умолчанию;
- Отправка сообщений электронной почты;
- Настройка утверждений для пользователей;
- Настройка сторонних поставщиков идентификационной информации.

Для пользовательского интерфейса используется вспомогательный NuGet-пакет: `Microsoft.AspNetCore.Identity.UI`.