---
share: true
tags:
 - NET/library
 - testing
aliases:
 - System.IO.Abstractions
---
# System.IO.Abstractions
Библиотека для тестирования операций ввода-вывода. Как написано в документации, "`System.Web.Abstractions` для файловой системы".

В основе библиотеки лежат `IFileSystem` и `FileSystem`. Вместо прямого вызова таких методов, как `File.ReadAllText`, используйте `IFileSystem.File.ReadAllText`. Они имеют точно такой же API, который можно внедрять и тестировать. Библиотека также поставляется с набором тестовых помощников, чтобы избавить вас от необходимости имитировать каждый вызов для базовых сценариев. Они не являются полной копией реальной файловой системы, но помогут вам в тестах.

## Ссылки
[GitHub](https://github.com/TestableIO/System.IO.Abstractions)
[NuGet](http://www.nuget.org/packages/System.IO.Abstractions)