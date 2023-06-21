---
share: true
tags:
 - NET/SignalR
 - CORS
---
# Что такое CORS и почему он важен
Подключение политик [[cors|CORS]] является, возможно, одной из самых главных вещей, которые нужно сделать для того, чтобы защитить приложение ASP.NET Core.
CORS применяется не только к SignalR, но ко всему приложению ASP.NET Core и всем его конечным точкам HTTP, включая SignalR[^1].
Здесь мы приведём небольшой пример задания политики CORS.
Перейдём к файлу **Program.cs** проекта **SignalRServer**. Добавим инициализацию промежуточного ПО и политики:
```csharp
builder.Services.AddCors(options => 
{
	options.AddPolicy("AllowAnyGet",
		builder => builder.AllowAnyOrigin()
			.WithMethods("GET")
			.AllowAnyHeader());

	options.AddPolicy("AllowExampleDomain",
		builder => builder.WithOrigins("https://example.com")
			.AllowAnyMethod()
			.AllowAnyHeader()
			.AllowCredentials());
});
```
Здесь мы добавили две политики. В первой, `AllowAnyGet`, мы разрешаем выполнять GET (и только GET) запросы с любыми заголовками с любого домена. Во второй, `AllowExampleDomain`, мы разрешаем выполнять запрос любого типа с любыми заголовками, а также передавать реквизиты пользователя, но только в том случае, когда запрос выполняется из источника `https://example.com`.
Теперь применим добавленные политики. Добавим вот этот код после вызова `app.UseRouting()`
```csharp
app.UseCors("AllowAnyGet").UseCors("AllowExampleDomain");
```
Важен порядок вызова методов `Use*`. Для `UseCors` важно, чтобы эти методы были вызваны, например, до вызова `app.UseResponseCaching()`.

[^1]: Тема разобрана [[ch-18-calling-web-apis-from-other-domains-using-cors|здесь]].
