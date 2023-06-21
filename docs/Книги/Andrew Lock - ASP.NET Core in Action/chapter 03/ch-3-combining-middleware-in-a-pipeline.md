---
aliases:
 - конвейер промежуточного ПО
 - конвейеру промежуточного ПО
share: true
tags:
 - NET/ASPNETCore/middleware
---
# Объединение компонентов в конвейер
У каждого компонента, по сути, есть одна основная задача — задача обработки только одного из аспектов запроса. Это позволяет как сохранить сами компоненты простыми и понятными, так и обеспечить необходимую гибкость при компоновке приложения.
Рассмотрим несколько простых сценариев.
## Простой сценарий конвейера 1: страница приветствия
При первой настройке приложения может быть полезным сперва использовать компонент для формирования страницы приветствия, чтобы убедиться, что запросы обрабатываются без ошибок.
Создадим конвейер, состоящий только из одного компонента — `WelcomePageMiddleware`. Для этого в методе `Configure` класса `Startup` вызовем метод `UseWelcomePage()` у объекта `IApplicationBuilder`. Вот какой класс у нас получился:
```csharp
public class Startup
{
	public void Configure(IApplicationBuilder app)
	{
		app.UseWelcomePage();
	}
}
```
Здесь `UseWelcomePage()` — это метод расширения. На самом деле вызывается более общий метод (тоже метод расширения, кстати):
```csharp
app.UseMiddleware<WelcomePageMiddleware>();
```
Согласно соглашению, все такие методы начинаются со слова `Use`.
Итак, мы добавили `WelcomePageMiddleware` в конвейер.
## Простой сценарий конвейера 2: обработка статических файлов
Теперь создадим простейший конвейер, возвращающий нам статические файлы из папки wwwroot. Воспользуемся методом расширения `UseStaticFiles`, чтобы добавить единственный компонент - `StaticFileMiddleware`.
```csharp
public class Startup
{
	public void Configure(IApplicationBuilder app)
	{
		app.UseStaticFiles();
	}
}
```
Что делает этот компонент? Он смотрит на путь запроса, и если запрашиваемый файл есть в папке wwwroot, возвращает его (вернее, его содержимое) в ответе. Если же файла нет, он передает запрос дальше. Если же *после* этого компонента других компонентов нет, то создастся ответ с кодом ошибки 404 (файл не найден) — ASP.NET Core всегда добавляет в в конец конвейера "фиктивный"  компонент, формирующий такой ответ.
## Простой сценарий 3: приложение со страницами Razor
Теперь создадим конвейер для страниц Razor. Проговорим еще раз: порядок, в котором вызываются методы Use в методе `Configure`, соответствует порядку компонентов в конвейере.
```csharp
public class Startup
{
	public void ConfigureServices(IServiceCollection services)
	{
		services.AddRazorPages(); //0
	}
	public void Configure(IApplicationBuilder app)
	{
		app.UseExceptionHandler("/Error"); //1
		app.UseStaticFiles(); //2
		app.UseRouting(); //3
		app.UseEndpoints(endpoints => //4
	 	{
			endpoints.MapRazorPages();
		})
	}
}
```
0. Инициализируем Razor Pages
1. Первый компонент — обработка ошибок
2. Второй компонент — компонент статических файлов
3. Третий компонент — компонент маршрутизации
4. Четвертый — компонент конечной точки. Если страницы Razor нет, конвейер вернет 404
Вот какой получился конвейер:
![[Pasted image 20211223201216.png]]
Чтобы чётче понимать важность порядка, попробуем добавить компонент `WelcomePageMiddleware`отвечающего на путь `/`.
Сперва добавим его *в начало* конвейера:
```csharp
public class Startup
{
	public void ConfigureServices(IServiceCollection services)
	{
		services.AddRazorPages();
	}
	public void Configure(IApplicationBuilder app)
	{
		app.UseWelcomePage("/");
		app.UseExceptionHandler("/Error");
		app.UseStaticFiles();
		app.UseRouting();
		app.UseEndpoints(endpoints =>
	 	{
			endpoints.MapRazorPages();
		})
	}
}
```
Теперь запрос к `/` никогда не достигнет компонента конечной точки — Welcome page сформирует ответ раньше.
Переместим вызов `app.UseWelcomePage("/")` в конец конвейера:
```csharp
public class Startup
{
	public void ConfigureServices(IServiceCollection services)
	{
		services.AddRazorPages();
	}
	public void Configure(IApplicationBuilder app)
	{
		app.UseExceptionHandler("/Error");
		app.UseStaticFiles();
		app.UseRouting();
		app.UseEndpoints(endpoints =>
	 	{
			endpoints.MapRazorPages();
		})
		app.UseWelcomePage("/");
	}
}
```
В этом случае Welcome Page не будет достигнут — путь `/` обработает компонент конечной точки (а на неизвестные ему пути Welcome Page не настроен).