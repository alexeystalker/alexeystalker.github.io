---
aliases:
 - startup.cs
share: true
tags:
 - NET/ASPNETCore
---
# Класс Startup: настройка вашего приложения
Класс `Startup` отвечает за настройку двух основных аспектов приложения:
- *регистрация сервисов* — [[aspnetcore-service|любые классы]], от которых зависит ваше приложение, должны быть зарегистрированы — настраивается в методе `ConfigureServices`;
- *промежуточное ПО и конечные точки* — как ваше приложение обрабатывает запросы и отвечает на них - настраивается в методе `Configure`.

Вот как выглядит примерная сигнатура класса `Startup`:
```csharp
public class Startup
{
	public void ConfigureServices(IServiceCollection services)
	{
		//Детали метода
	}

	public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
	{
		//Детали метода
	}
}
```
`IHostBuilder` , созданный в классе `Program`, вызывает метод `ConfigureServices`, затем `Configure`. Все сервисы, зарегистрированные в `ConfigureServices`, доступны в `Configure`.
Как мы видим, класс `Startup` не реализует никаких интерфейсов; методы вызываются с помощью *рефлексии (отражения, reflection)* при помощи поиска методов с предопределенными именами `ConfigureServices` и `Configure`.
## Добавление и настройка сервисов
Этот метод — точка конфигурации *контейнера зависимостей* `IServiceCollection`, с помощью которого реализуется шаблон *[[dependency-injection-pattern|внедрения зависимостей]] (Dependency Injection)*. 
Вот что будет в этом методе в нашем приложении:
```csharp
public class Startup
{
	public void ConfigureServices(IServiceCollection services)
	{
            services.AddRazorPages();
	}
}
```
Здесь `services` — это [[di-container|контейнер зависимостей]]. `AddRazorPages()` — метод расширения, объединяющий регистрацию всех сервисов, необходимых для работы Razor Pages.
## Определяем, как обрабатываются запросы с помощью промежуточного ПО
В последнем методе конфигурации класса `Startup`, `Configure`,  определяется конвеер промежуточного ПО для приложений, указывающий, как ваше приложение обработает HTTP-запросы. Вот метод `Configure` для шаблонного приложения:
```csharp
public class Startup
{
	public void Configure(
		IApplicationBuilder app,
		IWebHostEnvironment env)
	{
		if (env.IsDevelopment())
		{
			app.UseDeveloperExceptionPage();
		}
		else
		{
			app.UseExceptionHandler("/Error");
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
}
```
`IApplicationBuilder`, передаваемый параметру `Configure` используется для определения порядка, в котором выполняются компоненты. Порядок важен, потому что компонент может использовать только объекты, созданые предыдущим компонентом в конвейере.
Параметр `IWebHostEnvironment` здесь используется для обеспечения поведения, зависящего от окружения. При запуске в окружении разработки в конвейер добавляется один компонент для обработки исключений, а в промышленном — другой.
В целом объект `IWebHostEnvironment` содержит сведения о текущем окружении и предоставляет доступ к ряду свойств этого окружения, например:
- `ContentRootPath` — расположение рабочего каталога приложения
- `WebRootPath` — расположение папки wwwroot
- `EnvironmentName` — имя окружения

К моменту вызова класса `Startup` объект `IWebHostEnvironment` уже задан, и значения в нем доступны только для чтения.
В зависимости от окружения, добавляем в конвейер либо (для окружения разработки) `DeveloperExceptionPageMiddleware`для показа подробностей выпавшего исключения, либо `ExceptionHandlerMiddleware` для показа стандартного сообщения об ошибке без подробностей.
Следующий добавленный компонент — `HttpsRedirectionMiddleware` — гарантирует, что приложение отвечает только на HTTPS-запросы.
Далее добавляется `StaticFileMiddleware` для обработки запросов к статическим файлам. Он проверяет, есть ли запрашиваемый файл в папке wwwroot, и если да — возвращает его. Иначе управление передается следующему компоненту.
И,наконец, самое главное — компонент маршрутизации, авторизации и конечных точек. Компонент машрутизации использует URL-адрес, чтобы определить, какая страница запрошена, затем компонент авторизации решает, разрешен ли пользователю доступ к этой странице и затем метод `MapRazorPages` определяет, что нужно вернуть результат выполнения страницы Razor.

