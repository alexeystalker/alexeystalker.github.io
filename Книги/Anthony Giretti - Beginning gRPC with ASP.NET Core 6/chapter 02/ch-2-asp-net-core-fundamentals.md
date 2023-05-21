---
share: true
tags:
 - NET/ASPNETCore
---
# Основы ASP.NET Core
Точкой входа для любого приложения ASP.NET Core является файл **Program.cs**. Вот такой файл создаётся при использовании [[project-template|шаблона проекта]] ASP.NET Core 6:
```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", () => "Hello World!");

app.Run();
```
**Program.cs**[^1] состоит из двух частей:
 - *Конфигурация сервисов*[^2] - включает в себя настройку приложения, сторонних библиотек, аутентификации, авторизации и регистрацию сервисов в [[di-container|контейнере зависимостей]][^3];
 - *Активация сервисов*[^4] - определяет [[ch-3-combining-middleware-in-a-pipeline|конвейер промежуточного ПО]].

Конфигурация сервисов происходит *до* вызова метода `builder.Build()`, а активация - после и до вызова `app.Run()`. Вот ещё один пример конфигурации приложения Razor Pages:
```csharp
var builder = WebApplication.CreateBuilder();
//Конфигурация сервисов
builder.Services.AddRazorPages();

var app = builder.Build();
//Активация сервисов
if(!app.Environment.IsDevelopment())
{
	app.UseExceptionHandler("/Error");
	app.UseHsts();
}
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();
app.MapRazorPages();

app.Run()
```

В секции конфигурации сервисов у приложения есть доступ к системе конфигурации[^5], а затем конфигурацию можно передать через [[dependency-injection-pattern|внедрение зависимостей]].
```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Services.Configure<SmtpConfiguration>(Configuration.Section("SmtpConfiguration"));
//...
```

[^1]: По сравнению с [[ch-2-program-cs|предыдущей версией]], в ASP.NET Core 6 файл **Program.cs** заметно лаконичнее, и, к тому же, не требует **Startup.cs** 
[^2]:[[ch-2-startup-cs|Ранее]] за это отвечал метод `ConfigureServices()` класса `Startup`;
[^3]:Концепция внедрения зависимостей не изменилась с .NET5 и описана Э.Локом [[ch-10-service-configuration-with-dependency-injection|здесь]];
[^4]:[[ch-2-startup-cs|Ранее]] за это отвечал метод `Configure()` класса `Startup`;
[^5]:Конфигурирование практически не изменилось с .NET5, о нём можно прочитать [[ch-11-configuring-application|здесь]]