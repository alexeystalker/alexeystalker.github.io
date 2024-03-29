---
share: true
tags:
 - NET/ASPNETCore/endpoint
---
# Создание веб-сайта с помощью страниц Razor
Как правило, конвейер промежуточного ПО включает в себя компонент `EndpointMiddleware`, в котором (чаще всего) и содержится основная часть логики приложения. Также этот компонент служит точкой входа для пользователей приложения. Обычно принимает одну из трёх форм:
- *веб-приложение с HTML-разметкой, разработанное для непосредственного использования пользователями.* В этом случае страницы Razor обрабатывают запросы URL-адресов, получают данные, отправленные с помощью форм и генерируют HTML-код, используемый для просмотра данных и навигации по приложению;
- *API, предназначенный для использования на другой машине или в коде*. В этом случае приложение предоставляет данные в машиночитаемом формате, таком как JSON или XML;
- *веб-приложение с HTML-разметкой и API*. Возможно также, что приложение будет сочетать обе вышеупомянутые формы.

Далее рассмотрим, как ASP.NET Core использует Razor Pages для обработки первого из этих вариантов.
#### [[ch-4-intro-to-razor-pages|Введение в Razor Pages]]
#### [[ch-4-comparing-razor-pages-and-mvc|Сравнение Razor Page и MVC в ASP.NET Core]]
#### [[ch-4-razor-pages-handlers|Razor Pages и обработчики страниц]]