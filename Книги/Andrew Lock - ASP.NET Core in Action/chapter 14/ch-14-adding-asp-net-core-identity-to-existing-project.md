---
share: true
tags:
 - NET/ASPNETCore/Identity
---
# Добавляем ASP.NET Core Identity в существующий проект
Добавим пользователей в наше [[ch-12-sample-application|приложение рецептов]].
На этом этапе мы добавим ASP.NET Core Identity в существующее приложение, уже использующее EF Core.
Чтобы добавить Identity в приложение, нужно выполнить 4 шага:
1. Добавьте пакеты NuGet для ASP.NET Core Identity.
2. Настройте класс `Startup` для использования `AuthenticationMiddleware` и добавьте сервисы Identity в контейнер внедрения зависимостей.
3. Обновите модель данный EF Core.
4. Обновите страницы Razor Pages и макеты, чтобы предоставить ссылки на UI Identity.

Далее подробнее о каждом из шагов.
#### [[ch-14-configuring-asp-net-identity-services-and-middleware|Настройка сервисов ASP.NET Core Identity и промежуточного ПО]]
#### [[ch-14-updating-ef-core-data-model-to-support-identity|Обновление модели данных EF Core для поддержки Identity]]
#### [[ch-14-updating-razor-views-to-link-to-identity-ui|Обновление представлений Razor для связи с пользовательским интерфейсом Identity]]