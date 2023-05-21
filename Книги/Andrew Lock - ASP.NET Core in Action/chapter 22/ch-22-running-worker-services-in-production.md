---
share: true
tags:
 - NET/worker-service
 - Windows/service
 - Linux/systemd
---
# Запуск сервисов рабочей роли в промышленном окружении
Очень часто воркеры разворачивают в промышленном окружении в виде службы Windows или демона Linux. В этом разделе рассмотрим, как добавить поддержку служб Windows или systemd для Linux.
Для добавления поддержки нужно добавить следующие пакеты NuGet:
- *Microsoft.Extensions.Hosting.Systemd* — добавляет поддержку для запуска приложения в виде приложения systemd. Добавляет метод расширения `UseSystemd()` для `IHostBuilder`;
- *Microsoft.Extensions.Hosting.WindowsServices* — добавляет поддержку запуска приложения в качестве службы Windows. Добавляет метод `UseWindowsService()` для `IHostBuilder`.

Вот пример использования `UseWindowsService()`:
```csharp
public class Program
{
	public static void Main(string[] args)
	{
		CreateHostBuilder(args).Build().Run();
	}
	
	public static IHostBuilder CreateHostBuilder(string[] args) =>
		Host.CreateDefaultBuilder(args)
			.ConfigureServices((hostContext, services) =>
			{
				//Настраиваем воркер как обычно
				services.AddHostedService<Worker>();
			})
			.UseWindowsService();
}
```
Итак, чтобы установить приложение в качестве службы Windows, нужно:
1. добавить пакет Microsoft.Extensions.Hosting.WindowsServices;
2. добавить вызов метода `UseWindowsService()`;
3. [[ch-16-running-vs-publishing|опубликовать приложение]];
4. в командной строке с правами администратора установить приложение при помощи команды `sc`:
	```bash
	sc create "My Test Service" BinPath="C:\path\to\MyService.exe"
	```
5. Теперь можно управлять службой из панели управления службами. также можно управлять службой при помощи команды `sc`, например, чтобы запустить службу, выполните `sc start "My Test Service"`, а чтобы удалить - `sc delete "My Test Service"`.

> [!warning] Предупреждение
> Мы указали минимум шагов, необходимых для установки службы Windows. При работе в промышелнном окружении необходимо учитывать дополнительные аспекты безопасности. [Дополнительные сведения в документации Microsoft](https://learn.microsoft.com/aspnet/core/host-and-deploy/windows-service?view=aspnetcore-6.0&tabs=visual-studio)

Отметим, что точно также можно развернуть и приложение ASP.NET Core.

---
Для установки в качестве демона systemd можно выполнить аналогичный процесс, установив пакет Microsoft.Extensions.Hosting.Systemd и вызвав метод `UseSystemd()`. Подробнее о настройке systemd [здесь](https://learn.microsoft.com/aspnet/core/host-and-deploy/linux-nginx?view=aspnetcore-6.0#monitor-the-app).