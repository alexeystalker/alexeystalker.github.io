---
share: true
tags:
 - NET/ASPNETCore/middleware
---
# Добавление в конвейер с помощью метода Use
Для добавления компонентов в конвейер можно использовать метод расширения `Use()`.
Как и в [[ch-19-simple-endpoints-with-run-extension|случае]] с `Run()`, в методе `Use()` указывается лямбда-функция, которая выполняется, когда запрос доходит до компонента. В функцию передаются два параметра:
- `HttpContext`, *предоставляющий текущий запрос и ответ*;
- *указатель на остальную часть конвейера в виде* `Func<Task>`.

Используя указатель, можно контролировать, будет ли выполняться остальная часть конвейера, и, если будет, то когда. Такой подход делает доступным различные сценарии. Например. можно реализовать проверку работоспособности в едином компоненте, а не [[ch-19-branching-middleware-pipelines-with-map-extension|при помощи ветвления]]:
```csharp
public void Configure(IApplication app)
{
	app.Use(async (HttpContext context, Func<Task> next) =>
	{
		if (context.Request.Path.StartsWithSegments("/ping"))
		{
			context.Response.ContentType = "text/plain";
			await context.Response.WriteAsync("pong");
		}
		else
		{
			await next();
		}
	});
	
	app.UseStaticFiles();
}
```

> [!warning] Внимание!
> *Не изменяйте* объект `Response` после вызова метода `next()`! Кроме того, нельзя вызывать метод `next()`, если производилась запись в `Response.Body`; запись в этот поток может начать Kestrel для передачи потокового ответа браузеру. Обычно можно безопасно *считывать* данные из `Response`, например `StatusCode` или `ContentType` ответа.

Еще один вариант использования `Use()` — изменение каждого запроса или ответа, проходящего через него. Например, если мы хотим добавлять в каждый ответ некие заголовки безопасности, такие как [[ch-18-enforcing-https-for-your-whole-app|HSTS]] (или иные)[^1].

[^1]: Есть специальный сайт для проверки сайта на заголовки безопасности, [https://securityheaders.com](https://securityheaders.com). Также на этом сайте можно получить информацию о том, какие заголовки следует добавлять и почему. 

Предположим, мы хотим добавить такой заголовок, `X-Content-Type-Options: nosniff`. Используем метод расширения `Use()` для перехвата запроса и добавления заголовка в каждый ответ.
```csharp
public void Configure(IApplicationBuilder app)
{
	app.Use(async (HttpContext context, Func<Task> next) =>
	{
		context.Response.OnStarting(() =>
		{
			context.Response.Headers["X-Content-Type-Options"] = "nosniff";
			return Task.CompletedTask;
		});
		await next();
	});
	
	app.UseStaticFiles();
	app.UseRouting();
	app.UseEndpoints(endpoints => endpoints.MapControllers());
}
```
Здесь мы регистрируем callback-функцию, которую Kestrel вызовет перед отправкой ответа в браузер. Такой подход гарантирует, что заголовок не окажется случайно удалён другим компонентом, находящимся далее по конвейеру, а также защищает от ситуации изменения заголовков после того, как ответ уже начал передаваться в браузер.