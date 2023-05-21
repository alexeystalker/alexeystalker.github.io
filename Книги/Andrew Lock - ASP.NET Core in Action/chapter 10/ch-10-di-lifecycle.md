---
share: true
tags:
 - NET/ASPNETCore
 - NET/DI
---
# Жизненный цикл: когда создаются зависимости?
Всякий раз, когда у контейнера запрашивается зарегистрированный сервис, он может делать одно из двух:
- создать и вернуть новый экземпляр сервиса;
- вернуть существующий экземпляр сервиса.

[[dependency-injection-pattern#Жизненный цикл сервиса|Жизненный цикл]] сервиса управляет поведением контейнера в отношении этих двух параметров.
В ASP.NET Core есть три разных типа жизненного цикла при регистрации во встроенном контейнере:
- *Tranisent* — каждый раз, когда запрашивается сервис, создается новый экземпляр;
- *Scoped* — в пределах области применения все запросы сервиса будут предоставлять один и тот же объект. Для разных областей будут возвращены разные объекты. В ASP.NET Core каждый веб-запрос получает свою область применения;
- *Singleton* — возвращается один и тот же экземпляр, независимо от области видимости.

Для иллюстрации каждого из типов ЖЦ построим следующий пример. Пусть есть  класс `DataContext`:
```csharp
public class DataContext
{
	static readonly Random _rand = new Random();
	public DataContext()
	{
		RowCount = _rand.Next(1, 1_000_000_000);
	}
	
	public int RowCount { get; }
}
```
В нем разные экземпляры будут возвращать разные числа `RowCount`, но один экземпляр будет возвращать всегда одно и то же число.
Далее, пусть есть класс `Repository`, зависящий от `DataContext`:
```csharp
public class Repository
{
	private readonly DataContext _dataContext;
	public Repository(DataContext dataContext)
	{
		_dataContext = dataContext;
	}
	public int RowCount => _dataContext.RowCount;
}
```
Здесь `RowCount` возвращает значение `RowCount` экземпляра `DataContext`.
Наконец, есть Razor Page `RowCountModel`, зависящая от обоих этих классов:
```csharp
public class RowCountModel : PageModel
{
	private readonly Repository _repository;
	private readonly DataContext _dataContext;
	public RowCountModel(Repository repository, DataContext dataContext)
	{
		_repository = repository;
		_dataContext = dataContext;
	}
	
	public void OnGet()
	{
		DataContextCount = _dataContext.RowCount;
		RepositoryCount = _repository.RowCount;
	}
	
	public int DataContextCount { get; set; }
	public int RepositoryCount { get; set; }
}
```
При визуализации страницы будем выводить значения, сложенные в `DataContextCount` и `RepositoryCount`.
Таким образом, для каждого запроса будет строиться `RowCountModel`, для чего из [[di-container|контейнера зависимостей]] будут запрашиваться `Repository` и `DataContext`. Для двух запросов будет следующая картина:
![[Pasted image 20220322203823.png]]
Рассмотрим результат для каждого из типов ЖЦ:
## Transient
Зарегистрируем сервисы как Transient:
```csharp
services.AddTransient<DataContext>();
services.AddTransient<Repository>();
```
Теперь каждый раз контейнер будет создавать новый экземпляр. Следовательно, `DataContext`, внедренный в `Repository` будет отличаться от внедренного в `RowCountModel`.
Результат двух запросов страницы:
![[Pasted image 20220322204454.png]]
## Scoped
Жизненный цикл типа Scoped предполагает, что один экземпляр объекта будет использоваться в пределах заданной области. В ASP.NET Core область применения соответствует запросу, поэтому в рамках одного запроса контейнер будет использовать один и тот же объект. Очень часто с таким ЖЦ регистрируют сервисы, которые должны быть ограничены рамками запроса, такие как контексты БД или сервис аутентификации.
Зарегистрируем `DataContext` как scoped:
```csharp
services.AddScoped<DataContext>();
```
Результат:
![[Pasted image 20220322205251.png]]
## Singleton
Экземпляр сервиса создаётся при первой необходимости, и затем используется во всё время работы приложения. Особенно полезен этот ЖЦ для объектов, не имеющих состояния, либо для объектов, содержащих данные для совместного использования в запросах, например кэшей. Важно помнить, что код такого сервиса должен быть потокобезопасным.
Зарегистрируем `DataContext` как singleton:
```csharp
services.AddSingleton<DataContext>();
```
Вызовем `RowCountModel` дважды:
![[Pasted image 20220322210042.png]]

## Следите за захваченными зависимостями
Предположим, мы настраиваем ЖЦ для `DataContext` и `Repository`:
- `DataContext` — scoped, так как он должен использоваться для одного запроса
- `Repository` — singleton, так как он потокобезопасный и не имеет собственного состояния.

Если мы поступим таким образом, мы получим *захваченную зависимость*. `DataContext`, использованный в `Repository` всегда будет одним и тем же, так как он будет сохраняться всё время жизни `Repository`.
Поэтому сервис должен использовать только те зависимости, жизненный цикл которых превышает или эквивалентен ЖЦ сервиса. Singleton сервис может использовать только singleton зависимости. Scoped сервис — только scoped и singleton. Transient — все три вида.

По умолчанию ASP.NET Core проверяет захватываемые зависимости на соответствие этому правилу только в окружении разработки, так как она влияет на производительность. Чтобы включить или отключить проверку независимо от окружения, нужно задать параметр `ValidateScopes` при создании `HostBuilder`:
```csharp
public class Program
{
	public static void Main(string[] args)
	{
		CreateHostBuilder(args).Build().Run();
	}
	
	public static IHostBuilder CreateHostBuilder(string[] args) =>
		Host.CreateDefaultBuilder(args)
		.ConfigureWebHostDefaults(webBuilder =>
		{
			webBuilder.UseStartup<Startup>();
		})
		.UseDeafultServiceProvider(options =>
		{
			options.ValidateScopes = true;
			options.ValidateOnBuild = true;
		})
}
```
В примере также включен параметр `ValidateOnBuild`, который отвечает за проверку всего [[dependency-graph|графа зависимостей]] на наличие регистрации всех требуемых зависимостей.