---
share: true
tags:
 - NET/ASPNETCore/publish
---
# Оптимизация клиентских ресурсов с помощью BundlerMinifier
Страница Razor, сгенерированная приложением, часто содержит в себе ссылки на дополнительные файлы CSS, JavaScript, а также картинки и другие ресурсы. Браузер должен загрузить все эти файлы, чтобы правильно отобразить страницу.
Вот как происходит загрузка страницы Account/Login:
![[Pasted image 20220716185434.png]]
Особенно ярко это проявляется в промышленном окружении, когда приложением пользуются, используя интернет-соединения, различные как по скорости, так и по устойчивости.
Время, необходимое для загрузки приложения, в основном зависит от двух факторов:
- *общий размер ответов*
- *количество запросов*. В HTTP/1.0 и HTTP/1.1 можно сделать не больше 6 одновременных запросов к серверу — остальные должны дождаться загрузки предыдущих. В [[http-2|HTTP/2]], который поддерживается Kestrel, такого ограничения нет, но нельзя быть уверенным, что он используется клиентами.

Есть много возможностей для ускорения загрузки страниц. Два самых простых из них - использовать упаковку и [[minification|минификацию]] кода, чтобы уменьшить количество и размер запросов.
> [!tip] Определение
> *Упаковка (bundling)* - процесс объединения нескольких файлов в один с целью уменьшить количество запросов.

## Ускорение работы приложения с помощью упаковки и минификации кода
К примеру, для [[ch-12-sample-application|приложения с рецептами]] запрос страницы Login приведет к 10 запросам к серверу:
- один запрос HTML;
- три запроса файлов CSS;
- шесть запросов файлов JavaScript.

Некоторые из этих файлов являются стандартными библиотеками, а некоторые — файлами конкретного приложения. Мы можем уменьшить число файлов CSS, применив упаковку при публикации. Также можно сократить размер файлов JavaScript при помощи минификации.
## Добавляем BundlerMinifier в приложение
Обычно для минификации и упаковки используют средства типа Grunt или Webpack, однако ASP.NET Core может выполнять эту задачу с помощью пакета NuGet — BuildBundlerMinifier, или расширения Visual Studio — Bundler & Minifier. Их не обязательно использовать, если вы уже пользуетесь другими инструментами, но в случае нового проекта их вполне можно попробовать.
Добавить инструмент в проект можно двумя способами:
- установить расширение VS: **Tools > Extensions and Updates > Bundler & Minifier**;
- добавить в проект Nuget-пакет BuildBundlerMinifier.

Воспользуемся вторым способом:
```bash
dotnet add package BuildBundlerMinifier
```
Теперь при каждом выполнении `dotnet build` будет происходить упаковка и минификация, сконфигурированные в файле bundleconfig.json.
Пример bundleconfig.json:
```json
[
	{
		"outputFileName": "wwwroot/css/site.min.css",
		"inputFiles": [
			"wwwroot/css/site.css",
			"wwwroot/css/navigation.css"
		]
	},
	{
		"outputFileName": "wwwroot/js/site.min.js",
		"inputFiles": [
			"wwwroot/js/site.js",
			"wwwroot/js/*.js",
			"!wwwroot/js/site.min.js"
		],
		"minify": {
			"enabled": true,
			"renameLocals": true
		},
		"sourceMap": false
	}
]
```
Для каждого пакета указываем набор файлов для включения, а также дополнительные опции. Допускаются wildcards, а также можно указать файлы, которые не надо включать, при помощи `!` в начале имени файла.
## Использование минифицированных файлов с помощью тег-хелпера окружения
Оптимизированные файлы хорошо использовать в промышленном окружении, тогда как в окружении разработки целесообразно использовать исходные файлы.
[[ch-8-environment-tag-helper|Ранее]] мы видели, как можно применить тег-хелпер окружения для показа баннера, когда приложение работало в тестовом окружении. Аналогичный подход можно использовать и для того, чтобы переключаться между оптимизированными и неоптимизированными файлами:
```html
<environment include="Development">
	<link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.css" />
	<link rel="stylesheet" href="~/css/navigation.css" />
	<link rel="stylesheet" href="~/css/site.css" />
</environment>
<environment exclude="Development">
	<link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.min.css" />
	<link rel="stylesheet" href="~/css/site.min.css" />
</environment>
```
## Обслуживание часто используемых файлов из CDN
Несколько преимуществ использовать CDN (Content Delivery Network, сеть доставки содержимого) для доставки часто используемых файлов, типа Bootstrap или JQuery:
- обычно они быстрые;
- они избавляют сервер от необходимости предоставлять файл, экономя пропускную способность;
- поскольку файл предоставляется с другого сервера, он не учитывается в шести одновременных запросах, которые разрешено делать к серверу в протоколах HTTP/1.0 и HTTP/1.1[^1];
- множество разных веб-приложений могут ссылаться на один и тот же файл, поэтому есть вероятность, что браузер пользователя уже закэшировал его.

Основная проблема при использовании CDN — он может выйти из строя, и тогда браузер не сможет загрузить его. Необходимо предусмотреть резервный механизм. Например, можно воспользоваться тег-хелпером `asp-fallback-test*` для проверки, загрузился ли нужный файл:
```html
<environment include="Development">
	<link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.css" />
	<link rel="stylesheet" href="~/css/navigation.css" />
	<link rel="stylesheet" href="~/css/site.css" />
</environment>
<environment exclude="Development">
	<link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css"
		  asp-fallback-test-class="sr-only"
		  asp-fallback-test-property="position"
		  asp-fallback-test-value="absolute"
		  asp-fallback-href="~/lib/bootstrap/dist/css/bootstrap.min.css" />		  
	
	<link rel="stylesheet" href="~/css/site.min.css" />
</environment>
```

[^1]: Заметка о [предельных числах запросов](https://docs.pushtechnology.com/cloud/latest/manual/html/designguide/solution/support/connection_limitations.html) в различных браузерах.