---
share: true
tags:
 - NET/ASPNETCore/routing
---

# Получение значений маршрута из HttpContext
Обычно получение [[route-value|значений маршрута]] происходит через [[model-binding|привязку модели]]. Однако иногда необходимо получить эти значения напрямую из [[httpcontext|контекста]].
> [!Note] Например
> В SignalR Core можно указать для хаба [[route-template|шаблон маршрута]], но стандартные механизмы типа атрибута `[BindProperty]` при этом не работают.

Есть два способа: через `context.GetRouteData().Values` или `context.GetRouteValue("min")`, а именно:
```csharp
var routeValues = context.GetRouteData().Values;
var valueStr = routeValues["paramName"] as string;
//--- или ---
var valueStr2 = context.GetRouteValue("paramName") as string;
```

## Ссылки
https://andrewlock.net/accessing-route-values-in-endpoint-middleware-in-aspnetcore-3/