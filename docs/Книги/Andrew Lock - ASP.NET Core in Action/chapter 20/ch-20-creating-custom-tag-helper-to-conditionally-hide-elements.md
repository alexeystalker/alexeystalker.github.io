---
share: true
tags:
 - NET/Razor/tag-helper
---
# Создание специального тег-хелпера для условного скрытия элементов
Для того, чтобы проконтролировать отображение элемента в шаблоне Razor, обычно используют подобную конструкцию:
```razor
@{
	var showContent = true;
}
@if(showContent)
{
	<p>The content to show</p>
}
```
Однако на практике использование такой конструкции может оказаться неудобным, так как усложняет использование редакторов HTML, а переключение между C# и HTML может быть утомительным.
Создадим тег-хелпер `if`, чтобы избежать этой проблемы. Тег-хелпер будет применяться в качестве атрибута:
```razor
@{
	var showContent = true;
}
<p if="showContent">The content to show</p>
```
Вот код такого тег-хелпера:
```csharp
[HtmlTargetElement(Attributes = "if")]
public class IfTagHelper: TagHelper
{
	[HtmlAttributeName("if")]
	public bool RenderContent { get; set; } = true;
	
	public override void Process(
		TagHelperContext context, TagHelperOutput output)
	{
		if(RenderContext == false)
		{
			output.TagName = null;
			output.SuppressOutput();
		}
	}
	
	public override int Order => int.MinValue;
}
```
Здесь:
- задавая значение для свойства `Attributes` в атрибуте `HtmlTargetElement`, мы гарантируем, что тег-хелпер будет запускаться атрибутом `if`. Это позволяет применять тег-хелпер к любому HTML-элементу с этим атрибутом;
- `[HtmlAttributeName("if")]` связывает атрибут `if` со свойством `RenderContent`;
- в случае `RenderContent == false` мы удаляем элемент, записывая в `output.TagName` значение `null`;
- используем `output.SupressOutput()` для запрещения отрисовки содержимого элемента с атрибутом;
- переопределяем свойство `Order` - это гарантирует нам, что `IfTagHelper` будет выполнен первым в случае, если к элементу применено несколько тег-хелперов.

> [!Note] Примечание
> Важно не забыть [[ch-20-printing-environment-info-with-custom-tag-helper#Регистрация тег-хелпера|зарегистрировать]] свои тег-хелперы в файле **\_ViewImports.cshtm**l при помощи директивы `@addTagHelper`.