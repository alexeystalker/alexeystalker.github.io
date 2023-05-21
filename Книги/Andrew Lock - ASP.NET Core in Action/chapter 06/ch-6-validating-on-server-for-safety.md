---
share: true
tags:
 - NET/ASPNETCore/validation
---
# Валидация модели на сервере в целях безопасности
Валидация [[binding-model|модели привязки]] происходит до выполнения обработчика страницы, однако обработчик выполняется независимо от результата валидации. Обработчик страницы должен проверить результат.
Razor Pages сохраняет выходные данные попытки валидации модели в свойстве `PageModel.ModelState`. Это объект класса `ModelStateDictionary`, содержащий список возникших ошибок валидации и некоторые служебные свойства.
Пример.
```csharp
public class CheckoutModel : PageModel
{
	[BindProperty]
	public UserBindingModel Input { get; set; }
	
	public IActionResult OnPost()
	{
		if(!ModelState.IsValid)
		{
			return Page();
		}
		/*Сохранение в БД, обновление пользователя*/
		return RedirectToPage("Success");
	}
}
```
Если появились ошибки валидации, мы просто показываем пользователю страницу, дополненную сообщениями об ошибках. Сообщения можно настраивать, задавая свойство `ErrorMessage` атрибута. Например, для `[Required]`: `[Required(ErrorMessage="Required")]`.
![[Pasted image 20220206214942.png]]
В случае успешной валидации мы перенаправляем пользователя на страницу с сообщением об успехе. Это пример реализации шаблона [[prg-pattern|POST-REDIRECT-GET]].
