---
share: true
tags:
 - NET/Razor/tag-helper
---
# Тег-хелпер раскрывающегося списка
Помимо полей `<input>` в веб-формах часто присутствуют поля `<select>` для обеспечения выбора одного или нескольких вариантов из списка.
Чтобы использовать элементы `<select>` в коде Razor, необходимо включить в PageModel два свойства: одно для списка отображаемых значений, и другое для хранения значения (или значений).
Пример:
```csharp
public class SelectListsModel : PageModel
{
	[BindProperty]
	public InputModel Input { get; set; }
	public IEnumerable<SelectListItem> Items { get; set; } = new List<SelectListItem>
	{
		new SelectListItem { Value = "csharp", Text = "C#" },
		new SelectListItem { Value = "python", Text = "Python" },
		new SelectListItem { Value = "cpp", Text = "C++" },
		new SelectListItem { Value = "java", Text = "Java" },
		new SelectListItem { Value = "js", Text = "JavaScript" },
		new SelectListItem { Value = "ruby", Text = "Ruby" }
	};
	
	public class InputModel
	{
		public string SelectedValue1 { get; set; }
		public string SelectedValue2 { get; set; }
		public IEnumerable<string> MultiValues { get; set; }
	}
}
```
Здесь мы видим, что для заполнения списка значениями мы должны использовать тип `IEnumerable<SelectListItem>`, важно понимать, что тег-хелпер работает только с коллекцией с элементами этого типа.
Тег-хелпер предоставляет атрибуты `asp-for` для свойства, к которому нужно выполнить привязку, и `asp-items` для передачи элементов списка. Если в `asp-items` нужно отобразить элементы перечисления, разумно воспользоваться специальным помощником: `asp-items="Html.GetEnumSelectList<TEnum>()"`.
Пример шаблона Razor:
```razor
@page
@model SelectListsModel
<select asp-for="Input.SelectedValue1" asp-items="Model.Items"></select>
<select asp-for="Input.SelectedValue2" asp-items="Model.Items" size="4"></select>
<select asp-for="Input.MultiValues" asp-items="Model.Items"></select>
```
---
Помимо этого в поле `<select>` элементы могут быть сгруппированы при помощи элемента `<optgroup>.
Вот что нам нужно добавить в наш код (отбросим для краткости определение `InputModel` и сократим число опций):
```csharp
public class SelectListsModel : PageModel
{
	[BindProperty]
	public InputModel Input { get; set; }
	public IEnumerable<SelectListItem> Items { get; set; }
	
	public SelectListsModel
	{
		var dyn = new SelectListGroup { Name = "Dynamic" };
		var stat = new SelectListGroup { Name = "Static" };
		Items = new List<SelectListItem>
		{
			new SelectListItem {
				Value = "js",
				Text = "JavaScript",
				Group = dyn
			},
			new SelectListItem {
				Value = "cpp",
				Text = "C++",
				Group = stat
			},
			new SelectListItem {
				Value = "python",
				Text = "Python",
				Group = dyn
			},
			new SelectListItem { Value = "csharp", Text = "C#" },
		};
	}
}
```
Теперь при необходимости будут генерироваться элементы `<optgroup>`.
Шаблон
```razor
@page
@model SelectListsModel
<select asp-for="Input.MultiValues" asp-items="Model.Items"></select>
```
сгенерирует следующий HTML-код:
```html
<select id="Input_MultiValues" name="Input.MultiValues" multiple="multiple">
	<optgroup label="Dynamic">
		<option value="js">JavaScript</option>
		<option value="python">Python</option>
	</optgroup>
	<optgroup label="Static">
		<option value="cpp">C++</option>
	</optgroup>
	<option value="csharp">C#</option>
</select>
```
---
Еще одно распространенное требование — иметь возможность указать, что значение не было выбрано, иначе по умолчанию будет выбрано первое значение в списке.
Для этого нужно либо добавить значение *Не выбран* к доступным объектам `SelectListItem`, либо вручную добавить элемент прямо в шаблоне Razor:
```html
<select asp-for="SelectedValue" asp-items="Model.Items">
	<option value="">**Не выбран**</option>
```
