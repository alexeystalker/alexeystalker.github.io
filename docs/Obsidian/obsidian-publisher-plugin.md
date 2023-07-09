---
tags: [Obsidian/plugin, github]
share: true
---
# Паблишер для Obsidian
Опенсорсный паблишер для Обсидиана. Заточен для работы с Github Pages.
Состоит из двух основных частей:
- Плагина для Obsidian;
- Шаблона для репозитория на GitHub с настроенными Actions для генерации статического сайта из .md файлов;
- Генератора статических сайтов из Markdown MkDocs.

В теории, плагин для Obsidian можно использовать независимо от шаблона, с любым другим репозиторием. Полезно, если есть желание использовать другой Markdown генератор.

## Поддерживаются
Копия списка из документации. Рядом мой комментарий.
-    Wikilinks (`[[Links]]`) — можно преобразовывать в Markdown ссылки плагином, или оставлять так, и тогда mkdocs превратит их в ссылки самостоятельно;
-    File transclusion/embed, both wikilinks and markdown links — не использую embed карточек, не могу сказать;
-    Obsidian callout and custom callout — врезки (callouts) работают;
-    Folder notes and their citation
-    Custom attributes
-    Sharing state and custom folder hierarchy
-    Mobile and desktop
-    File mini preview on Hover — пока не разобрался, как включить.

## Не поддерживаются
- Обратные ссылки (в идеале - некий список карточек, ссылающихся на открытую карточку)

## Ссылки
https://obsidianmkdocs.github.io/obsidian_mkdocs_publisher_docs/