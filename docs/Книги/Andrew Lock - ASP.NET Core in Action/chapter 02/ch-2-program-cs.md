---
share: true
tags:
 - NET/ASPNETCore
---
# Класс Program: сборка веб-хоста
Приложения ASP.NET Core по сути являются консольными приложениями, поэтому запускаются также: с помощью файла Program.cs со стандартной точкой входа для консольных приложений `static void Main`.
В приложениях ASP.NET Core метод `Main` используется для создания и запуска экземпляра `IHost` — ядра приложения ASP.NET Core, содержащего конфигурацию приложения и сервер Kestrel.
Рассмотрим полный листинг класса Program.cs:
```csharp
public class Program
{
	public static void Main(string[] args) //1
	{
		CreateHostBuilder(args)
			.Build() //2
			.Run();  //3
	}

	public static IHostBuilder CreateHostBuilder(string[] args) =>
		Host.CreateDefaultBuilder(args) //4
			.ConfigureWebHostDefaults(webBuilder => //5
			{
				webBuilder.UseStartup<Startup>(); //6
			});
}
```
Здесь:
1. Создаем `IHostBuilder` с помощью метода `CreateHostBuilder`.
2. Создаем и возвращаем экземпляр `IHost` из `IHostBuilder`.
3. Запускаем `IHost`, начинаем прослушивать запросы и генерировать ответы.
4. Создаем `IHostBuilder`, используя конфигурацию по умолчанию.
5. Настраиваем приложение для использования Kestrel и прослушивания HTTP-запросов.
6. Класс `Startup` определяет большую часть конфигурации приложения.

Большая часть настройки приложения происходит в `IHostBuilder`, создаваемом вызовом метода `CreateDefaultBuilder`, однако часть ответственности делегируется специальному классу [[ch-2-startup-cs|Startup]], используемому в дженерик-методе `UseStartup<Startup>`.
Почему существует такое разделение на `Program` и `Startup`? Рассмотрим картинку:
![[Pasted image 20211214193912.png]]
Как правило, `Program` — это место, где настраивается инфраструктура приложения — HTTP-сервер, интеграция с IIS, источники конфигурирования. А `Startup` — место определения компонентов и функций, используемых приложением, а также конвейера промежуточного ПО (middleware).
Классы `Program` у разных ASP.NET Core приложений обычно похожи, а вот `Startup` — могут сильно различаться. На протяжении жизненного цикла приложения `Program` меняется значительно реже, нежели `Startup`.
Итак, большая часть настройки нашего приложения происходит в двух методах — `CreateDefaultBuilder` и `ConfigureWebhostDefaults`, использующий объект `WebHostBuilder` для конфигруирования Kestrel.
После завершения настройки вызываем метод `Build` для создания экземпляра интерфейса `IHost`, и у него — метод `Run`, после которого начинается прослушивание HTTP-запросов.

> [!important] Важно!
> В данный момент концепция разделения точки входа на **Program.cs** и **Startup.cs** устарела. Начиная с NET6, используется гораздо более [[ch-2-asp-net-core-fundamentals|лаконичная схема]]