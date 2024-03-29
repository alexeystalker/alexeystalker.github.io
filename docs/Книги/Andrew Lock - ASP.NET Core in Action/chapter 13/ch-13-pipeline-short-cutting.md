---
share: true
tags:
 - NET/ASPNETCore/filters
---
# Прерывание выполнения конвейера
Чтобы прервать выполнение фильтров авторизации, ресурсов, действий, страниц и результатов, нужно задать для `context.Result` значение `IActionResult`.

Однако, как показано [[ch-13-mvc-filter-pipeline|здесь]] и [[ch-13-razor-pages-filter-pipeline|здесь]], конвейер фильтров не является полностью линейным, поэтому, прерывая его выполнение, не всегда можно сделать полный разворот назад. Так, например, прерывание фильров действий отменит только выполнение метода действия, но фильтры результатов и этапы выполнения результатов будут работать.

Также вопрос в том, что произойдет, если есть несколько фильтров одного типа, например, три фильтра ресурсов, и прерывание случится во втором из них?
Как показано на картинке, первый из выполненных фильтров также выполнит команду `*Executed()`.
![[Pasted image 20220526204701.png]]

В таблице указано, как влияет на выполнение конвейера прерывание каждого типа фильтров.

|Тип фильтра|Как прервать выполнение?|Что еще запускается?|
|---|---|---|
|Фильтры авторизации|Задать `context.Result`|Запускается только `IAlwaysRunResultFilter`|
|Фильтры ресурсов|Задать `context.Result`|Функции фильтров ресурсов `*Executed` из более ранних фильтров запускаются с `context.Cancelled = true`.<br>`IAlwaysRunResultFilter` запускается перед выполнением `IActionResult`|
|Фильтры действий|Задать `context.Result`|Игнорируется только выполнение метода действия. Фильтры действий, которые идут первыми в конвейере, выполняют свои методы `*Executed` с `context.Cancelled = true`, потом методы фильтров результатов, выполнения результатов, и фильтров ресурсов `*Executed` выполняются, как обычно|
|Фильтры страниц|Задать `context.Result` в `OnPageHandlerSelected`|Игнорируется только выполнение обработчика страниц. Фильтры страниц, которые идут первыми в конвейере, выполняют свои методы `*Executed` с `context.Cancelled = true`, потом методы фильтров результатов, выполнения результатов, и фильтров ресурсов `*Executed` выполняются, как обычно|
|Фильтры исключений|Задать `context.Result` и `Exception.Handled = true`|Выполняются все функции фильтров ресурсов `*Executed`. `IAlwaysRunResultFilter` запускается перед выполнением `IActionResult`|
|Фильтры результатов|Задать `context.Cancelled = true`|Фильтры результатов, находящиеся первыми в конвейере, выполняют свои методы `*Executed` c `context.Cancelled = true`. Все функции фильтров ресурсов `*Executed` работают как обычно|

