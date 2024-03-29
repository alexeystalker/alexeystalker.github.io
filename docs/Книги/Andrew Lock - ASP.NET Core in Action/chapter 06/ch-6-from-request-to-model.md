---
share: true
tags:
 - NET/ASPNETCore/binding-model
---
# От запроса к модели: делаем запрос полезным
ASP.NET Core обрабатывает запрос, выполняя [[page-handler|обработчик страницы]]. Обработчики страницы — обычные методы C#, поэтому должны вызываться обычным способом. Если у них есть параметры, они получают значения в процессе [[model-binding|привязки модели]].

Вот пример кода с привязкой модели к свойствам.
```csharp
public class IndexModel : PageModel
{
	[BindProperty]
	public string Category { get; set; }
	
	[BindProperty(SupportsGet = true)]
	public string UserName { get; set; }
	
	public void OnGet() {}
	
	public void OnPost(ProductModel model) {}	
}
```
Помеченные атрибутом `[BindProperty]` свойства будут привязаны к параметрам запросов POST и PUT, помеченные атрибутом `[BindProperty(SupportsGet = true)]` помимо POST и PUT будут привязаны к запросам GET.
По умолчанию ASP.NET Core использует три разных *источника привязки* при создании [[binding-model|моделей привязки]], причем использует их в указанном порядке.
1. *Значения формы* — отправляются в теле HTTP-запроса при отправке формы методом POST;
2. *значения маршрута* — получаются из [[segment|сегментов]] URL-адреса или значений по умолчанию после сопоставления маршрута;
3. *значения строки запроса* — передаются в конце URL-адреса и не участвуют в маршрутизации.

При этом используется первое найденное значение модели.
После привязки каждого свойства модель валидируется и передается в свойство объекта `PageModel` или в качестве параметра обработчику страницы.

#### [[ch-6-binding-simple-types|Связывание простых типов]]
#### [[ch-6-binding-complex-types|Привязка сложных типов ]]
#### [[ch-6-choosing-a-binding-source|Выбор источника привязки]]