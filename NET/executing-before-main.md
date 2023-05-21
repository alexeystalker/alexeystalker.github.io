---
share: true
tags: [NET]
---
# Выполнение кода перед Main
> [!info] Main
> Метод `Main` — это точка входа в приложение C#. При запуске приложения метод `Main` вызывается первым.[^1]

[^1]:[Documentation](https://docs.microsoft.com/en-us/dotnet/csharp/fundamentals/program-structure/main-command-line?WT.mc_id=DT-MVP-5003978)

На самом деле метод `Main` может быть не первым методом сборки, который будет выполняться при запуске приложения. Существуют различные методы, которые могут выполняться перед методом `Main`.
## Статический конструктор
Статический конструктор используется для инициализации любых статических данных или для выполнения определённого действия, которое необходимо выполнить только один раз. Он вызывается автоматически перед созданием первого экземпляра или обращением к любым статическим элементам. Поэтому статический конструктор вызовется перед методом `Main`.
```csharp
class Program
{
  static Program() =>
    Console.WriteLine("Program.cctor");
  static void Main() =>
    Console.WriteLine("Hello, World!");
}
```
## Инициализаторы модулей
Инициализаторы модулей позволяют библиотекам выполнять однократную инициализацию при загрузке без необходимости явного вызова пользователем чего-либо. Когда среда выполнения загружает модуль (DLL), она вызывает методы инициализации модуля перед выполнением любого кода из этого модуля.
```csharp
using System.Runtime.CompilerServices;
class Initializer
{
  // Статический конструктор выполняется
  // перед инициализаторами модуля
  static Initializer() 
    => Console.WriteLine("Init.cctor");

  [ModuleInitializer]
  public static void Initialize1() =>
    Console.WriteLine("Module Init 1");

  [ModuleInitializer]
  public static void Initialize2() =>
    Console.WriteLine("Module Init 2");
}
```
## Стартап хуки
Переменная среды `DOTNET_STARTUP_HOOKS` может использоваться для указания списка управляемых сборок, содержащих тип `StartupHook` с публичным статическим методом `Initialize()`:
```csharp
class StartupHook
{
  static StartupHook() =>
    Console.WriteLine("StartupHook.cctor");

  // Запустите приложение с переменной среды
  // DOTNET_STARTUP_HOOKS=<полный путь к сборке с этим классом>
  public static void Initialize() =>
    Console.WriteLine("Startup hook");
}
```

Каждый из этих методов будет вызываться в указанном порядке перед точкой входа `Main`:

```
Init.cctor
Module Init 1
Module Init 2
StartupHook.cctor
Startup hook
program.cctor
Hello, World!
```
## Ссылки
https://www.meziantou.net/executing-code-before-main-in-dotnet.htm
