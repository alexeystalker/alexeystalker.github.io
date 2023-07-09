---
share: true
tags: [NET/ASPNETCore]
---
# Список всех маршрутов в ASP.NET Core
Когда ваше приложение ASP.NET Core станет достаточно большим, вам может понадобиться полное представление обо всех маршрутах. Существует несколько способов объявления маршрутов. Вы можете использовать минимальные API, контроллеры, Razor Pages, gRPC, HealthCheck и т. д. Но все они используют одну и ту же систему маршрутизации под капотом.

Коллекция маршрутов может быть указана путем извлечения коллекции `EndpointDataSource` из DI.

Пример для минимальных API:
```csharp
if (app.Environment.IsDevelopment())
{
  app.MapGet("/debug/routes", 
   (IEnumerable<EndpointDataSource> sources) =>
    string.Join("\n", sources.SelectMany(s => s.Endpoints)));
}
```

Добавление этого блока в пример приложения из [документации](https://docs.microsoft.com/en-us/aspnet/core/tutorials/min-web-api) выведет следующие маршруты:

```
HTTP: GET /
HTTP: GET /todoitems
HTTP: GET /todoitems/complete
HTTP: GET /todoitems/{id}
HTTP: POST /todoitems
HTTP: PUT /todoitems/{id}
HTTP: DELETE /todoitems/{id}
HTTP: GET /debug/routes
```

Если используются полноценные контроллеры, `EndpointDataSource` можно получить из сервисов. Также можно использовать различные свойства полученных маршрутов, чтобы вывести нужную информацию:

```csharp
public string Get([FromServices] 
 IEnumerable<EndpointDataSource> sources)
{
  var sb = new StringBuilder();
  var endpoints = 
    sources.SelectMany(s => s.Endpoints);
  foreach (var e in endpoints)
  {
    sb.AppendLine(e.DisplayName);
 
    if (e is RouteEndpoint re)
      sb.AppendLine(re.RoutePattern.RawText); 
  }

  return sb.ToString();
}
```
## Ссылки
https://www.meziantou.net/list-all-routes-in-an-asp-net-core-application.htm
https://t.me/NetDeveloperDiary/1486
