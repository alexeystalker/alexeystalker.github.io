---
share: true
tags:
 - NET/Razor/tag-helper
---
# Создание форм с помощью тег-хелперов
Веб-приложение — это не только отображение данных, пусть и динамических. Однако, почти всегда частью функциональности веб-приложения является возможность ввода пользователем новых данных, либо изменение существующих. В случае с традиционными веб-приложениями данные отправляют при помощи веб-форм.
Для создания веб-форм ASP.NET Core предоставляет функцию под названием *тег-хелперы*.
#### [[ch-8-catering-to-editors-with-tag-helpers|Редакторы кода и тег-хелперы]]
Тег-хелперы могут выполнять множество функций, изменяя элементы HTML, к которым они применяются. Здесь будет рассказано только о нескольких самых распространненых тег-хелперах, однако их гораздо больше. К тому же, всегда можно [[ch-20-creating-custom-razor-tag-helper|реализовать собственные тег-хелперы]], или использовать опубликованные другими на NuGet или GitHub[^1]

[^1]:Например, пакет [Tag Helper](https://github.com/DamianEdwards/TagHelperPack)

#### [[ch-8-creating-forms-using-tag-helpers|Создание форм с помощью тег-хелперов]]

---
Помимо форм, тег-хелперы могут применяться везде, где необходимо смешать логику на стороне сервера с генерацией HTML-кода. Один из таких примеров — создание ссылок на другие страницы приложения с помощью генерации URL-адресов на основе маршрутизации. Для этого есть специальный тег-хелпер.
#### [[ch-8-anchor-tag-helper|Создание ссылок с помощью тег-хелпера anchor]]
---
Если вы обнаружите, что пишете повторяющийся код в своей разметке, скорее всего, кто-то уже написал тег-хелпер, который может помочь. Вот отличный пример использования тег-хелперов для уменьшения количества кода: 
#### [[ch-8-append-version-tag-helper|тег-хелпер обновления версии]].

---
Или вот еще пример. 
#### [[ch-8-environment-tag-helper|Использование условной разметки с помощью тег-хелпера окружения]].