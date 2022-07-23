---
tags: [NET]
share: true
---
# Быстрый старт с библиотекой CommandLineParser
```csharp
using CommandLine;

namespace GetStartedSample
{
  static class Program
  {
    static void Main(string[] args)
    {
      // (1) default options
      var result = Parser.Default.ParseArguments<Options>(args);
      // or (2) build and configure instance
      var parser = new Parser(with => with.EnableDashDash = true);
      var result = parser.ParseArguments<Options>(args);
      Console.WriteLine("Hello World.");
    }
  }
}
```
`Options` - POCO класс, размеченный примерно так:
```csharp
[Option('o',"option",HelpText = "Help text for option")]
public string Option { get; set; }
```
Также полезно сразу изучить пункт `Verbs` - это подкоманды в стиле git типа `git push`:
```csharp
static int Main(string[] args) =>
  Parser.Default.ParseArguments<AddOptions, CommitOptions, CloneOptions>(args)
    .MapResult(
      (AddOptions options) => RunAddAndReturnExitCode(options),
      (CommitOptions options) => RunCommitAndReturnExitCode(options),
      (CloneOptions options) => RunCloneAndReturnExitCode(options),
      errors => 1);
```

## Ссылки
[[library-commandlineparser|Библиотека CommandLineParser]]
https://github.com/commandlineparser/commandline/wiki/Getting-Started
https://github.com/commandlineparser/commandline/wiki/Verbs
