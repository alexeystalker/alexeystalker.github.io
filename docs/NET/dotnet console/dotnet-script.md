---
share: true
tags: [NET/dotnet, script]
---
# Инструмент сценариев dotnet-script
`dotnet-script` - инструмент для исполнения сценариев в среде .NET Core.

Для начала установим инструмент:
```bash
dotnet tool install -g dotnet-script
```

Создадим файл скрипта helloworld.csx со следующим содержимым:
Console.WriteLine("Hello world!");

Теперь его можно выполнить:
```console
> dotnet script helloworld.csx
Hello world!
```
Для Linux или OSX нужно будет указать в первой строке стандартный заголовок, говорящий о том, какая программа может исполнять этот скрипт:
```bash
#!/usr/bin/env dotnet-script
```
Заголовок аналогичен файлам bash, python и т.п.

Скрипты могут иметь классы и подпрограммы:
```csharp
using System.Collections.Generic;
using System.Linq;

foreach (var i in Fibonacci().Take(20))
{
  Console.WriteLine(i);
}

private IEnumerable<int> Fibonacci()
{
  int current = 1, next = 1;

  while (true) 
  {
    yield return current;
    next = current + (current = next);
  }
}
```
Если нужно, вы можете обратиться к NuGet пакету внутри скрипта, используя синтаксис `#r` от Roslyn:
```csharp
#r "nuget: AutoMapper, 6.1.0"
Console.WriteLine("Hello AutoMapper!");
```
Также имея `dotnet-script`, установленный как глобальная утилита, как мы сделали выше, вы можете использовать его как REPL (интерактивную среду) прямо в консоли
```con
>dotnet script
> 2+2
4
> var x = "dotnet script";
> x.ToUpper()
"DOTNET SCRIPT"
```

## Ссылки
https://t.me/NetDeveloperDiary/1615
https://www.hanselman.com/blog/c-and-net-core-scripting-with-the-dotnetscript-global-tool
