---
share: true
tags:
 - NET/Razor/tag-helper
---
# Вывод информации об окружении с помощью специального тег-хелпера
Создадим специальный тег-хелпер для вывода системной информации в макет. Вызвать его из Razor можно, создав элемент `<system-info>` в шаблоне:
```html
<footer>
	<system-info></system-info>
</footer>
```
> [!tip] Совет
> Вероятно, мы не захотим показывать такую информацию в промышленном окружении, поэтому стоит использовать тег-хелпер `<environment>`, как показано [[ch-8-environment-tag-helper|тут]].

Самый простой способ создать тег-хелпер — унаследоваться от класса `TagHelper` и переопределить функцию `Process()` или `ProcessAsync()`:
```csharp
public class SystemInfoTagHelper : TagHelper
{
	private readonly HtmlEncoder _htmlEncoder;
	public SystemInfoTagHelper(HtmlEncoder htmlEncoder)
	{
		_htmlEncoder = htmlEncoder;
	}
	
	[HtmlAttributeName("add-machine")]
	public bool IncludeMachine { get; set; } = true;
	
	[HtmlAttributeName("add-os")]
	public bool IncludeOS { get; set; } = true;
	
	public override void Process(
		TagHelperContext context,
		TagHelperOutput output)
	{
		output.TagName = "div";
		output.TagMode = TagMode.StartTagEndTag;
		var sb = new StringBuilder();
		if (IncludeMachine)
		{
			sb.Append(" <strong>Machine</strong> ");
			sb.Append(_htmlEncoder.Encode(Environment.MachineName));
		}
		if (IncludeOS)
		{
			sb.Append(" <strong>OS</strong> ");
			sb.Append(_htmlEncoder.Encode(RuntimeInformation.OSDescription));
		}
		output.Content.SetHtmlContent(sb.ToString());
	}
}
```
Имя класса тег-хелпера определяет имя элемента. Здесь используется `SystemInfoTagHelper`, и по умолчанию отбрасывается суффикс (`TagHelper`), а вместо [PascalCase](https://ru.wikipedia.org/wiki/CamelCase) используется [lisp-case](https://ru.wikipedia.org/wiki/Snake_case) (он же kebab-case), что даёт нам `<system-info>`.
> [!tip] Совет
> Для более тонкого контроля за именем элемента можно использовать атрибут `[HtmlTargetElement]` c желаемым именем, например `[HtmlTargetElement("env-info")]`, чтобы получить `<env-info>`. HTML-теги нечувствительны к регистру!

Далее, мы внедрили в наш класс `HtmlHelper`, чтобы кодировать в HTML наши данные; как было [[ch-18-defending-against-xss-attacks|показано]], необходимо использовать `HtmlHelper`, чтобы избежать возможных XSS-атак.

Также мы определили два свойства: `IncludeMachine` и `IncludeOS`. Они декорированы атрибутом `[HtmlAttributeName]`, что позволяет задавать свойства из шаблона Razor, при этом в VS будет работать IntelliSence и проверка типа значения для этих свойств.

Теперь перейдём к методу `Process()`, который вызывается движком Razor для выполнения тег-хелпера. В этом методе мы определяем тип тега для отрисовки (`<div>`), нужно ли визуализировать начальный и конечный теги (или самозакрывающийся тег — зависит от типа тега) и HTML-содержимое тега `<div>`. HTML-содержимое задаётся через метод `Content.SetHtmlContent()`.
> [!Attention] Внимание!
> Перед записью в тег с помощью `SetHtmlContent()` контент необходимо кодировать в HTML! Альтернативой будет использование метода `SetContent()` который кодирует вывод автоматически.

## Регистрация тег-хелпера
Прежде чем использовать новый тег-хелпер, его надо зарегистрировать в файле [[ch-7-running-code-with-viewstart-and-viewimports|его надо зарегистрировать в файле]] **\_ViewImports.cshtml**, используя директиву `@addTagHelper`, указав полное имя класса тег-хелпера и сборки, например:
```razor
@addTagHelper CustomTagHelpers.SystemInfoTagHelper, CustomTagHelpers
```
Можно зарегистрировать все тег-хелперы из определенной сборки:
 ```razor
@addTagHelper *, CustomTagHelpers
```

 