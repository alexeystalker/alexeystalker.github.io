---
share: true
tags: [NET/WebApi]
---
# Привязка модели из нескольких источников в один класс в ASP.NET Core
Допустим, у нас есть следующий [[action-method|метод действия]] в ASP.NET API:
```csharp
public IActionResult Post(
 [FromBody] 
 RequestBody body,
 [FromHeader(Name = "Accept")]
 string accept,
 [FromHeader(Name = "X-Correlation-Id")]
 string correlationId,
 [FromQuery(Name = "filter")]
 string filter)
{
  // какая-то обработка
  return Ok();
}
```

Мы хотели бы объединить все параметры в один класс, `PostRequest`, вместо того чтобы использовать кучу разных. Создадим класс, который будет хранить все наши данные:
```csharp
public class PostRequest
{
  [FromBody]
  public RequestBody Body { get; set; };

  [FromHeader(Name = "Accept")]
  public string Accept { get; set; };

  [FromHeader(Name = "X-Correlation-Id")]
  public string CorrelationId { get; set; };

  [FromQuery(Name = "filter")]
  public string Filter { get; set; };
}

public class RequestBody
{
  public string SomeString { get; set; };
  public bool SomeBool { get; set; }
}

```
Изменим наш метод действия, чтобы он принимал наш тип:
```csharp
[HttpPost]
public IActionResult Post(PostRequest req)
{
  // просто возвращаем объект
  return new OkObjectResult(req);
}
```
Теперь, если мы попытаемся вызвать наш метод действия:
```http
POST
Accept: application/json
X-Correlation-Id: my-correlation-id

https://localhost:7001/home?filter=one

{"someString": "Some string", "someBool": true}
```
Мы получим следующее сообщение об ошибке:
```json
{
 …
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": {
    "Body": ["The Body field is required."],
    "Accept": ["The Accept field is required."],
    "filter": ["The Filter field is required."],
    "X-Correlation-Id": ["The CorrelationId field is required."]
  }
}
```
По умолчанию ASP.NET Core не осуществляет привязку модели из различных источников к одному классу. Чтобы изменить это поведение, нужно настроить `ApiBehaviourOptions` в методе `ConfigureServices()`:
```csharp
services.Configure<ApiBehaviorOptions>(options =>
{
  options.SuppressInferBindingSourcesForParameters = true;
});
```
Теперь, выполнив тот же запрос, что и раньше, мы получим правильный ответ:
```json
{
  "body": {
    "someString": "Some string",
    "someBool": true
  },
  "accept": "application/json",
  "correlationId": "my-correlation-id",
  "filter": "one"
}
```
## Ссылки
https://josef.codes/model-bind-multiple-sources-to-a-single-class-in-asp-net-core/

