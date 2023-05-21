---
share: true
tags:
 - NET/Razor/tag-helper
---
# Создание тег-хелпера для преобразования Markdown в HTML
В этом разделе напишем тег-хелпер, изменяющий своё содержимое, а именно — преобразует содержимое Markdown в содержимое HTML. Для этого используем [[library-markdig|библиотеку Markdig]].

Планируемый тег-хелпер `<markdown>` предполагается использовать как-то так:
```razor
@page
@model IndexModel

<markdown>
## This is a markdown title

This is a markdown list:

* Item 1
* Item 2

<div if="showContent">
	Content shown when showContent is true
</div>
</markdown>
```
Тег-хелпер будет визуализировать содержимое при помощи следующих шагов:
1. Визуализирует любое содержимое Razor внутри тег-хелпера, включая выполнение любых вложенных тег-хелперов и кода C# внутри тег-хелперов (в примере использовали [[ch-20-creating-custom-tag-helper-to-conditionally-hide-elements|созданный ранее]] `IfTagHelper`);
2. Преобразовывает полученную строку в HTML;
3. Заменяет содержимое визуализированным HTML и удаляет тег `<markdown>`.

Вот код простого подхода к реализации:
```csharp
public class MarkdownTagHelper: TagHelper
{
	public override async Task ProcessAsync(
		TagHelperContext context, TagHelperOutput output)
	{
		var markdownRazorContent = 
			await output.GetChildContentAsync(NullHtmlEncoder.Default);
		
		var markdown = 
			markdownRazorContent.GetContent(NullHtmlEncoder.Default);
			
		var html = Markdig.Markdown.ToHtml(markdown);
		
		output.Content.SetHtmlContent(html);
		output.TagName = null;
	}
}
```

В результате работы тег-хелпера (с учётом того, что `showContent = true`) будет таким:
```html
<h2>This is a markdown title</h2>
<p>This is a markdown list:</p>
<ul>
	<li>Item 1</li>
	<li>Item 2</li>
</ul>
<div>
	Content is shown when showContent is true
</div>
```
