---
share: true
tags:
 - NET/ASPNETCore/HTTPS
 - HTTPS
---
# Делаем так, чтобы протокол HTTPS использовался для всего приложения
В наши дни требуется практически принудительное использования HTTPS для всего веб-сайта. Помимо соображений безопасности можно упомянуть, что благодаря протоколу [[http-2|HTTP/2]] добавление TLS может улучшить производительность приложения.

Существует несколько подходов сделать так, чтобы HTTPS использовался для всего приложения.

> [!Note] Примечание
> Описанные в разделе меры применяются, в первую очередь, для приложений Razor Pages. Веб-API может просто отклонять HTTP запросы. [Дополнительно](https://docs.microsoft.com/aspnet/core/security/enforcing-ssl?view=aspnetcore-6.0&tabs=visual-studio)

Один из таких подходов — использовать *заголовки безопасности HTTP*[^1].
## Принудительное использование HTTPS с помощью заголовков HTTP Strict Transport Security
По умолчанию браузеры загружают приложения по протоколу HTTP. Следовательно, приложения должны поддерживать и HTTP, и HTTPS. Один из способов смягчить это заключается в добавлении заголовка HSTS.

> [!tip] Определение
> *HTTP Strict Transport Security (HSTS)* — заголовок, указывающий браузеру использовать HTTPS для всех *последующих* запросов к приложению. Его можно отправлять только лишь с ответами на HTTPS запросы. Также использование этого заголовка не влияет на обмен данными между серверами и актуально только для запросов из браузера[^2].

Заголовки HSTS рекомендуются для приложений в промышленном окружении, однако нет смысла для использования его в окружении разработки, так как непросто (а иногда и невозможно) отключить использование HTTPS для сайта, после того, как он был "включён" при помощи HSTS.
ASP.NET Core поставляется с промежуточным ПО для установки заголовков HSTS. Вот как использовать его в Startup.cs:
```csharp
public class Startup
{
	public void ConfigureServices(IServiceCollection services)
	{
		services.AddRazorPages();
		services.AddHsts(options => 
		{
			options.MaxAge = TimeSpan.FromHours(1);
		});
	}
	
	public void Configure(IApplication app, IWebHostEnvironment env)
	{
		if(env.IsProduction())
		{
			app.UseHsts(); //HstsMiddleware должен идти в самом начале конвейера
		}
		
		app.UseStaticFiles();
		app.UseRouting();
		app.UseAuthorization();
		app.UseEndpoints(endpoints => 
		{
			endpoints.MapRazorPages();
		});
	}
}
```
> [!tip] Совет
> В примере показано, как изменить значение `MaxAge` заголовка HSTS. Лучше начать с небольшого значения, чтобы потом, убедившись, что всё хорошо, увеличить его.

Обычно `HstsMiddleware` следует использовать совместно с компонентом, перенаправляющим все запросы с HTTP на HTTPS.

## Переадресация с HTTP на HTTPS с помощью компонента HttpsRedirectionMiddleware
ASP.NET Core поставляется с компонентом `HttpsRedirectionMiddleware`, который можно использовать для принудительного использования HTTPS в приложении. 
Если запрос доходит до `HttpsRedirectionMiddleware`, выполнение запроса прерывается, выполняя перенаправление на HTTPS версию запроса[^3]. Перенаправление по умолчанию происходит с использованием HTTP кода `307 Temporary Redirect`.
Вот как его обычно используют:
```csharp
public void Configure(IApplication app, IWebHostEnvironment env)
{
	app.UseExceptionHandler("/Error");
	if(env.IsProduction())
	{
		app.UseHsts();
	}

	app.UseHttpsRedirection();
	app.UseStaticFiles();
	app.UseRouting();
	app.UseAuthorization();
	app.UseEndpoints(endpoints => 
	{
		endpoints.MapRazorPages();
	});
}
```

HTTP запросы будут перенаправляться в первую сконфигурированную точку HTTPS. Если требуется перенаправление на другой порт (о котором знает Kestrel), его можно указать, используя переменную окружения `ASPNETCORE_HTTPS_PORT`[^4].
Если приложение не настроено на использование HTTPS, вместо перенаправление будет логироваться предупреждение вида
```log
Failed to determine the https port for redirect.
```

[^1]: [Статья о заголовках безопасности](https://scotthelme.co.uk/hardening-your-http-response-headers/)
[^2]: [Подробнее о HSTS](https://scotthelme.co.uk/hsts-the-missing-link-in-tls/)
[^3]: Видно, что, несмотря на использование HSTS и перенаправления, *один* HTTP запрос всё равно выполняется. Единственный способ избежать этого — предварительно загрузить заголовки HSTS.
[^4]: [Подробнее о настройке](https://docs.microsoft.com/en-us/aspnet/core/security/enforcing-ssl?view=aspnetcore-5.0&tabs=visual-studio)