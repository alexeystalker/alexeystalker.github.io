---
share: true
tags: [useful-tool]
---
# Набор утилит Microsoft PowerToys
Microsoft представляет набор утилит PowerToys (разной степени полезности). На данный момент в набор входят:
- [Always on Top](https://learn.microsoft.com/ru-ru/windows/powertoys/always-on-top) — позволяет устанавливать окно программы поверх всех других окон. Хоткей по умолчанию — `Win+Ctrl+T`;
- [Awake](https://learn.microsoft.com/en-us/windows/powertoys/awake) — утилита, позволяющая компьютеру не засыпать, не меняя плана электропитания;
- [Color Picker](https://learn.microsoft.com/en-us/windows/powertoys/color-picker) — утилита, выбирающая цвет (“пипетка”) и добавляющая его в предпочтительном формате в буфер обмена. Хоткей по умолчанию — `Win+Shift+C`;
- [FancyZones](https://learn.microsoft.com/en-us/windows/powertoys/fancyzones) — утилита для размещения окон на экране согласно активному шаблону (особенно удобна владельцам UltraWide мониторов). Есть несколько способов активировать режим выбора места для окна в активном шаблоне. При нажатии хоткея открывается режим редактирования шаблонов и выбора активного шаблона; хоткей по умолчанию — ``Win+Shift+` ``;
-  [File Locksmith](https://learn.microsoft.com/en-us/windows/powertoys/file-locksmith) — расширение оболочки, позволяющее проверить, какими процессами используется файл; вызывается из контекстного меню интересующего файла;
- [Аддоны для File Explorer](https://learn.microsoft.com/en-us/windows/powertoys/file-explorer) — набор дополнений, позволяющих просматривать содержимое файлов `.svg`, `.md`, `.pdf`, `.gcode` и файлов исходного кода популярных языков в панели предпросмотра (Preview Pane). Для файлов `json` и `xml` есть возможность форматирования при предпросмотре (без изменения исходного файла). Также добавляется возможность предпросмотра в тамбнейлах (thumbnails) для `.svg`, `.pdf`, `.gcode` и `.stl` файлов;
- [Редактор файла hosts](https://learn.microsoft.com/en-us/windows/powertoys/hosts-file-editor) — позволяет редактировать файл hosts (кто знает, тот поймёт). Вызывается из панели PowerToys;
- [Image Resizer](https://learn.microsoft.com/en-us/windows/powertoys/image-resizer) — утилита для изменения размера картинок, в том числе массового. Вызывается из контекстного меню картинки или группы картинок;
- [Keyboard Manager](https://learn.microsoft.com/en-us/windows/powertoys/keyboard-manager) — утилита для переопределения клавиш и сочетаний клавиш, при этом переопределения могут быть как глобальными, так и действующими в конкретном приложении. Об ограничениях и особенностях использования см. документацию по ссылке;
- [Mouse Utitlities](https://learn.microsoft.com/en-us/windows/powertoys/mouse-utilities) — несколько утилит для работы с мышью, вернее, с курсором мыши:
	- Find My Mouse — при активации (по умолчанию — двойное нажатие левого `Ctrl`) подсвечивает текущее положение курсора мыши;
	- Mouse Highlighter — подсвечивает место клика. Хоткей по умолчанию — `Win+Shift+H`;
	- Mouse Jump — позволяет переместить курсор на значительное расстояние (“перепрыгнуть”). Появляется маленькое поле выбора точки прыжка и после клика курсор перепрыгивает на выбранное место. Хоткей по умолчанию — `Win+Shift+D`;
	- Mouse Pointer Crosshairs — добавляет перекрестье (!) с курсором мыши в центре. Хоткей по умолчанию — `Win+Alt+P`;
- [Mouse Without Borders](https://learn.microsoft.com/en-us/windows/powertoys/mouse-without-borders) — утилита, позволяющая управлять несколькими (до четырёх) компьютерами при помощи одной мыши/клавиатуры, разделять буфер обмена и передавать файлы;
- [Paste as Plain Text](https://learn.microsoft.com/en-us/windows/powertoys/paste-as-plain-text) — утилита, предоставляющая возможность вставлять текст, находящийся в буфере обмена, без форматирования (удобно для пользователей MS Office, и, возможно, других офисных пакетов, которые при копировании копируют и форматирование текста). Хоткей по умолчанию — `Win+Ctrl+Alt+V`;
- [Peek](https://learn.microsoft.com/en-us/windows/powertoys/peek) — программа для быстрого предпросмотра файлов в отдельном окне. Почему-то не всегда корректно работает. Хоткей по умолчанию — `Ctrl+Space`;
- [PowerRename](https://learn.microsoft.com/en-us/windows/powertoys/powerrename) — программа для группового перенаименования файлов. Поддерживает регулярные выражения. Вызывается из контекстного меню группы;
- [PowerToys Run](https://learn.microsoft.com/en-us/windows/powertoys/run) — “строка запуска на стероидах”. Помимо обычного поиска и запуска программ может осуществлять поиск и действия с различными элементами внутри некоторых приложений при помощи плагинов, таких как, например, оснастка “Сервисы” или редактор реестра. Плагины могут вызываться принудительно при использовании команды активации, задаваемой в настройках. Хоткей по умолчанию — `Alt+Space`;
- [Quick Accent](https://learn.microsoft.com/en-us/windows/powertoys/quick-accent) — утилита для быстрого выбора диакритических знаков у букв. Включается по нажатию `←`, `→` или `Space` после нажатия и удерживания нужной буквы;
- [Registry Preview](https://learn.microsoft.com/en-us/windows/powertoys/quick-accent) — утилита для просмотра файлов `.reg`. Показывает содержимое в виде дерева ключей со значениями (как в редакторе реестра);
- [Screen Ruler](https://learn.microsoft.com/en-us/windows/powertoys/screen-ruler) — утилита для измерения различных элементов на экране (в пикселях). Хоткей по умолчанию — `Win+Shift+M`;
- [Shortcut Guide](https://learn.microsoft.com/en-us/windows/powertoys/shortcut-guide) — визуальный справочник хоткеев. Вызывается, в зависимости от настроек, хоткеем или долгим нажатием `Win`. Хоткей по умолчанию — `Win+Shift+/`;
- [Text Extractor](https://learn.microsoft.com/en-us/windows/powertoys/text-extractor) — утилита для распознавания текста из выделенной области на экране. Распознанный текст попадает в буфер обмена. Работает для языков, соответствующих установленным в системе языковым пакетам. О установке дополнительных пакетов см. справку по ссылке. Хоткей по умолчанию — `Win+Shift+T`;

## Ссылки
https://learn.microsoft.com/ru-ru/windows/powertoys/