---
share: true
tags:
 - NET/MVC
 - NET/WebApi
 - CORS
---
# Вызов веб-API из других доменов с помощью CORS
[[cors|Cross-origin resource sharing (CORS)]] — протокол, позволяющий JavaScript делать запросы из одного домена в другой.

Как мы [[ch-18-protecting-from-csrf-attacks|уже видели]], атаки CSRF могут быть довольно мощными, но они были бы еще опаснее, если бы не браузеры, реализующие *правило ограничения домена*. 

Однако, с ростом числа клиентских одностраничных приложений и отходом от монолитных приложений растёт потребность выполнять запросы к разным источникам. Чтобы решить эту проблему, существует веб-стандарт совместного использования ресурсов между разными источниками — CORS.
## Разбираемся с CORS и как он работает
Согласно веб-стандарту CORS, можно:
- разрешить запросы от `http://shopping.com` и `https://app.shopping.com`;
- разрешить запросы от разных источников, тоолько если используется метод `GET`;
- разрешить возвращать заголовок `Server` в ответах на запросы из разных источников;
- разрешить отправку учётных данных с запросами из разных источников.

Эти правила можно объединить в *политику* и применять разные политики к разным конечным точкам API.

CORS использует HTTP-заголовки. Когда веб-API получает запрос, он задаёт специальные заголовки для ответа, чтобы указать, разрешены ли запросы из разных источников, какие это источники и какие методы и заголовки могут использоваться в запросе.

Иногда перед отправкой реального запроса браузер отправляет *предварительный* запрос с использованием метода `OPTIONS`, чтобы проверить, разрешено ли сделать реальный запрос. Если API вернёт правильные заголовки, браузер отправит настоящий запрос.
![[Pasted image 20220821185812.png]]
[Спецификация CORS](https://fetch.spec.whatwg.org/#http-cors-protocol) довольно сложна, однако ASP.NET Core занимается деталями реализации сама. Нам нужно определить, кому и при каких обстоятельствах нужен доступ к API.
## Добавление глобальной политики CORS ко всему приложению
Чтобы добавить поддержку CORS в приложение, необходимо:
- добавить сервисы CORS в приложение;
- настроить хотя бы одну политику CORS;
- добавить промежуточное ПО CORS в конвейер;
- задать политику CORS по умолчанию для всего приложения, либо декорировать действия атрибутом `EnableCORS`, чтобы выборочно активировать CORS для определённых конечных точек.

Добавление сервисов CORS происходит через вызов `services.AddCors()` в `Startup.ConfigureServices`.

Основная часть усилий по настройке CORS уходит на конфигурирование политики.
Например, пусть наш API, размещенный на `http://api.shopping.com` был доступен из основного приложения на `http://shopping.com`. Нам нужно настроить API, разрешив запросы из разных источников.
> [!note] Примечание
> Именно основное приложение будет получать ошибки при попытке делать запросы из разных источников, но CORS нужно добавить *к API*, к которому требуется доступ, а *не* в основное приложение.

Настроим политику `AllowShoppingApp`, в которой разрешим запросы от `http://shopping.com`, а также использование люых HTTP-методов (без этого будут разрешены только GET, HEAD и POST):
```csharp
public void ConfigureServices(IServiceCollection services)
{
	services.AddCors(options =>
		{
			options.AddPolicy("AllowShoppingApp", policy =>
				policy.WithOrigins("http://shopping.com").AllowAnyMethod());
		});
	//...
}
```
> [!attention] Внимание
> У источников в методе `WithOrigins()` не должно быть завершающего слеша (`/`), иначе источники не совпадут и запросы будут отклонены.

Теперь добавим настроенную политику к приложению с помощью `CorsMiddleware`:
```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
	//...
	app.UseRouting();
	//***
	app.UseCors("AllowShoppingApp");
	//***
	app.UseAuthentication();
	app.UseAuthorization();
	
	app.UseEndpoints(endpoints => endpoints.MapControllers());
}
```
> [!note] Примечание
> Тут, как и в других случаях, важен порядок применения промежуточного ПО. Вызов `UseCors()` должен идти *после* метода `UseRouting()` и *до* метода `UseEndpoints()`. Обычно `UseCors()` размещается перед вызовом `UseAuthentication()`.

## Добавляем CORS к определенным действиям веб-API с помощью атрибута EnableCors
Блокировка запросов из разных источников по умолчанию — мера обеспечения безопасности. Поэтому, если мы точно знаем, что из разных источников будут обращаться только к некоторому подмножеству [[action-method|методов действия]] нашего API, нет смысла активировать CORS для всего приложения.
ASP.NET Core предоставляет атрибут `[EnableCors]`, позволяющий применить политику только к определенному контроллеру или методу действия. К разным контроллерам и методам могут быть применены разные политики.

Политики определяются, как показано выше, с помощью метода `AddPolicy()`; но вместо использования `UseCors()` с именем политики используем этот метод *без* указания конкретной политики. Чтобы применить конкретную политику к контроллеру или методу, используем `[EnableCors]` с именем политики. `[EnableCors]` для действия имеет приоритет над атрибутом для контроллера. Чтобы полностью исключить применение CORS для конкретного метода, используем атрибут `[DisableCors]`.
```csharp
[EnableCors("AllowShoppingApp")]
public class ProductController: Controller
{
	[EnableCors("AllowAnyOrigin")]
	public IActionResult GetProducts() { /* тело метода */ }
	
	public IActionResult GetProductPrice() { /* тело метода */ }
	
	[DisableCors]
	public IActionResult DeleteProduct() { /* тело метода */ }
}
```

## Настройка политик CORS
При помощи политик CORS можно настроить:
- источники, которые могут делать запросы;
- HTTP-методы, которые можно использовать;
- заголовки, которые может отправлять браузер;
- заголовки, которые браузер может прочитать из ответа;
- будет ли браузер отправлять с запросом учётные данные для ацтентификации.

Всё это можно определить при задании политик в вызове метода `AddCors()`.

Таблица доступных методов и их назначение:

|Пример метода CorsPolicyBuilder|Результат|
|---|---|
|`WithOrigins("http://shopping.com")`|Разрешает запросы от `http://shopping.com`|
|`AllowAnyOrigin()`|Разрешает запросы из любого источника.Это означает, что *любой* веб-сайт может делать JavaScript-запросы к API|
|`WithMethods()`/`AllowAnyMethod()`|Задает разрешенные HTTP-методы, с помощью которых можно выполнять запросы к API|
|`WithHeaders()`/`AllowAnyHeader()`|Задает заголовки, которые браузер может отправлять API. Если требуется ограничить заголовки, то необходимо включить по меньшей мере `"Accept"`, `"Content-Type"` и `"Origin"`, чтобы можно было выполнять действительные запросы|
|`WithExposedHeaders()`|Позволяет API отправлять в браузер дополнительные заголовки. По умолчанию в ответе отправляются только заголовки `Cache-Control`, `Content-Language`, `Content-Type`, `Expires`, `LastModified` и `Pragma`|
|`AllowCredentials()`|По умолчанию браузер не отправляет детали аутентификации при CORS запросах, пока это не будет явно разрешено. Также должна быть активирована отправка учётных данных на стороне клиента в JavaScript при создании запроса.|

