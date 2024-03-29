---
share: true
tags:
 - NET/ASPNETCore/binding-model
---
# Выбор источника привязки
По умолчанию связыватель модели ASP.NET Core использует три источника: данные формы, данные маршрута и строку запроса.
Иногда требуется специально указать, к какому источнику нужно выполнить привязку. Иногда требуется указать дополнительный источник, например, заголовок запроса.
В этом случае используются специальные атрибуты:
```csharp
public class PhotosModel : PageModel
{
	public void OnPost(
		[FromHeader] string userId,
		[FromBody] List<Photo> photos)
	{
		...
	}
}
```
Здесь мы указываем, что параметр `userId` нужно заполнить из HTTP-заголовка `userId`, а из тела запроса (по умолчанию в формате JSON) нужно заполнить список объектов `Photo`.
Вот возможные варианты источников привязки:
- `[FromHeader]` — привязка к значению HTTP-заголовка;
- `[FromQuery]` — привязка к значению строки запроса;
- `[FromRoute]` — привязка к значению маршрута;
- `[FromForm]` — привязка к данным формы из тела запроса;
- `[FromBody]` — привязка к содержимому тела запроса.

Отметим, что все атрибуты, за исключением `[FromBody]` можно использовать на нескольких свойствах или параметрах. Кроме того, при отправке формы атрибуты `[FromForm]` и `[FromBody]` фактически являются взаимоисключающими.

Также есть дополнительные атрибуты для настройки процесса привязки:
- `[BindNever]` — этот параметр будет всегда пропущен;
- `[BindRequired]` — если параметр не был указан или был пустым, добавится ошибка связывания модели;
- `[FromServices]` — используется, чтобы указать, что параметр должен быть предоставлен с применением [[dependency-injection-pattern|внедрения зависимостей]][^1].
- `[ModelBinder]` — атрибут, позволяющий указать точный источник, переопределить имя параметра и указать тип привязки.

[^1]:[[ch-10-injecting-services-into-am-ph-views|см. пример]]