---
share: true
aliases:
 - DTO
tags:
 - definition
---
# DTO - Data Transfer Object
*Объект передачи данных (Data Transfer Object, DTO)* - объект, используемый для инкапсуляции данных и их отправки из одной подсистемы приложения в другую. 
Чаще всего представляет собой [[./poco-class|POCO]]-объект. Как следует из названия, DTO используется, когда необходимо передать данные, например по сети. Не следует путать с объектом класса [[../Книги/Andrew Lock - ASP.NET Core in Action/definitions/domain-model|доменной модели]], более того, следует избегать излишних конвертаций из объекта доменной модели в DTO и обратно.

## Ссылки
[Ответ на Stack Overflow](https://stackoverflow.com/a/1058186)
[Мартин Фаулер и его мнение о месте применения DTO](https://martinfowler.com/bliki/LocalDTO.html)