---
share: true
tags:
 - NET/Razor/tag-helper
---
# Использование условной разметки с помощью тег-хелпера окружения
Во многих случаях необходимо визуализировать разный HTML-код в зависимости от того, работает ли веб-сайт в окружении разработки или в промышленном окружении.
Часто необходимо добавить баннер при запуске в тестовом окружении. Можно воспользоваться оператором `if`. Например, пусть в переменной `env` лежит текущее окружение.
```razor
@if(env = "Testing" || env = "Staging")
{
	<div class="warning">You are currently on a testing environment</div>
}
```
Однако, есть способ лучше — воспользоваться поставляемым `EnvironmentTagHelper`:
```html
<environment include="Testing, Staging">
	<div class="warning"> You are currently on a testing environment</div>
</environment>
```