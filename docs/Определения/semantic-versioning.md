---
aliases:
 - semver
 - семантическое версионирование
tags: [versioning, numeration]
share: true
---
# Семантическая нумерация версий
Согласно спецификации семантического версионирования, номер версии всегда должен состоять из следующих частей `MAJOR.MINOR.PATCH`, где каждая из частей инкрементируется в следующих случаях:
+ `MAJOR` — при внесении в API/пакет несовместимого изменения
+ `MINOR` — при внесении в API/пакет изменения с сохранением обратной совместимости
+ `PATCH` — при исправлении ошибки с сохранением обратной совместимости

Иногда добавляется 4 часть с номером билда. Сейчас обычно подставляют инкрементирующийся номер автоматического билда, однако (как в старых версиях VS) этот номер мог быть и случайным.

## Ссылки
https://semver.org/
