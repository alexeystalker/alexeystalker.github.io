---
share: true
tags:
 - NET/Razor
---
# Использование макетов для общей разметки
Файлы макета — по больщей части обычные шаблоны Razor, содержащие разметку, общую для нескольких страниц. Может быть несколько макетов, и макеты могут ссылаться на другие макеты.
Макеты обычно помещаются в папку **Pages/Shared**. Существует соглашение об использовании имени **\_Layout.cshtml** для файла базового макета. Обычно к файлам макета добавляется подчёркивание (`_`), чтобы отличить их от обычных шаблонов.
Файл макета похож на обычный шаблон, но в отличие от шаблона должен вызывать функцию `@RenderBody()`.
Вот пример:
```razor
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8" />
		<title>@ViewData["Title"]</title>
		<link rel="stylesheet" href="~/css/site.css" />
	</head>
	<body>
		@RenderBody()
	</body>
</html>
```
Здесь мы видим, что необходимые для HTML-файла элементы, такие как `<html>` и `<head>`, а также нужные на каждой странице `<title>` и `<link>`. Также виден способ передачи данных через `ViewData`.

Представления могут указать, какой файл макета использовать, задав свойство `Layout` внутри кодового блока Razor.
```razor
@{
	Layout = "_Layout";
	ViewData["Title"] = "Home Page";
}
<h1>@ViewData["Title"]</h1>
<p>This is the home page</p>
```

Содержимое представления будет отображаться там, где вызвана функция `@RenderBody()`. Следующий пример - результат объединения макета и представления
```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8" />
		<title>Home Page</title>
		<link rel="stylesheet" href="~/css/site.css" />
	</head>
	<body>
		<h1>Home Page</h1>
		<p>This is the home page</p>
	</body>
</html>
```