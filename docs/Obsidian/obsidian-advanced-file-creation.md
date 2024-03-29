---
share: true
tags:
 - Obsidian
---
# Продвинутый сценарий создания файлов в Obsidian
Давайте рассмотрим реальный сценарий использования плагинов в Obsidian — быстрое создание заметок по шаблону. Чтобы было проще, реализуем простой инструмент для записи расходов на топливо.
## Сценарий
Сценарий у нас будет такой. Пользователь приезжает на заправку, заправляет машину либо определенным числом литров, либо на определенную сумму и создаёт запись об этом, также указывая текущий пробег ТС. При этом хотелось бы сократить число необходимых для этого действий.
## Шаблон файла
Используем плагин [[obsidian-templater-plugin|Templater]] для шаблонизации. Рассмотрим шаблон:
```
<%*
var expenseType = "Заправка"; //заготовка для других типов записей
var vehicle = await tp.system.suggester(["Автомобиль", "Мотоцикл"], ["Автомобиль", "Мотоцикл"]);
var litres = await tp.system.prompt("Литры");
litres = litres == null ? 0 : litres;
var price = await tp.system.prompt("Цена за литр");
price = price == null ? 0 : price;
var total = 0;
if (litres > 0 && price > 0) {
	total = litres * price;
} else if ((litres > 0 && price == 0) || (litres == 0 && price > 0)) {
	total = await tp.system.prompt("Общая стоимость", 0);
}
if (total > 0) {
	if (litres == 0) {
		litres = total / price;
	} else if (price == 0) {
		price = total / litres;
	}
}
%>---
expenseType: <% expenseType %>
vehicle: <% vehicle %>
---
litres::<% litres %>
price::<% price %>
total::<% total %>
mileage::<% tp.system.prompt("Пробег") %>
```
Здесь мы воспользовались возможностью написания *скриптов* на JS, которые Templater выполняет при создании файла из шаблона. Такой скрипт должен быть заключён в стандартные для Templater командные символы с добавлением звёздочки: `<%* ... %>`. Также мы воспользовались возможностью ввода значений в модальных окнах. Сперва мы покажем модальное окно с вариантами выбора ТС (вдруг у нас неколько транспортных средств, и мы, разумеется, захотим указать, какое ТС мы заправляем в данный момент). Затем мы покажем последовательно окно ввода количества залитых литров топлива и цены за литр. При этом мы можем *не ввести* какое-либо из этих значений, и тогда появится окно для ввода общей стоимости, а незаполненное значение будет рассчитано на основе введённых данных. Если же ввели и число литров, и цену за литр — общая стоимость будет рассчитана и ввод этого значения не потребуется. После выполнения скрипта мы расставим полученные значения по местам, а затем покажем модалку для ввода пробега, так как пробег не участвует в вычислениях.
Также я добавил поле `expenseType`, если в будущем мы захотим записывать не только заправки, но и другие расходы. После этого все записанные значения можно использовать для каких-то агрегатов, используя плагин [[obsidian-dataview-plugin|Dataview]] или другие плагины на его основе.
Значения для `expenseType` и `vehicle` я проставил во frontmatter, а остальные — в inline-полях, но, кажется, это дело вкуса, Dataview одинаково (в данном случае) работает с обоими типами значений.
## Команда для создания файла
Воспользуемся ещё одной возможностью Templater — командой для применения шаблона. Для этого сперва создадим ещё один шаблон вот с таким содержимым:
```
<%*
const template = tp.file.find_tfile("templates/vehicleExpenseTemplate");
const filename = tp.date.now("YYYY-MM-DD-HH-mm");
const folder = app.vault.getAbstractFileByPath("periodic/vehicleExpenses");
await tp.file.create_new(template, filename, true, folder);
%>
```
Здесь скрипт находит файл шаблона (который мы создали в предыдущем разделе), и папку с записями трат, формирует имя файла из текущей даты по шаблону (такой шаблон я выбрал для удобства сортировки созданных файликов), и затем создаёт файл в папке на основе шаблона со сформированным именем, при этом выполнится скрипт из выбранного шаблона. Теоретически здесь мы тоже можем использовать модальные окна для пользовательского ввода (например, чтобы на основе него выбрать нужный шаблон файла или нужную папку), но в данный момент не станем это делать.
После того как файл создан, идём в настройки Templater-а и находим раздел **Template Hotkeys**
![[Pasted image 20230824163039.png|Pasted image 20230824163039.png]]
и, нажав на кнопку, добавляем вызов нашего шаблона с командой. Тонкость в том, что, несмотря на то, что раздел называется Template Hotkeys, хоткей добавлять не обязательно, главное, что при этом создаётся *команда Obsidian*, и её можно найти и вызвать, например, при помощи *Палитры команд*:
![[Pasted image 20230824163345.png|Pasted image 20230824163345.png]]
## Кнопка для команды
В принципе, команду, описанную в предыдущем разделе, можно вызывать и так, из Палитры, но это неудобно в нашем сценарии, когда пользователь находится на заправке и пытается сделать запись в мобильном Обсидиане.
Поэтому воспользуемся ещё одним плагином — [[obsidian-commander-plugin|Commander]]. Это плагин для добавления любых команд из Палитры в различные меню Обсидиана. Добавим при помощи него нашу *команду* в Ленту (Ribbon):
![[Pasted image 20230824164208.png|Pasted image 20230824164208.png]]
Я воспользовался иконкой машинки, но можно, разумеется, использовать любую другую.

Всё, теперь по нажатию кнопки появится ряд модальных окошечек, после чего откроется созданный файлик со всеми данными.

## Ссылки
[Документация Templater](https://silentvoid13.github.io/Templater/introduction.html)