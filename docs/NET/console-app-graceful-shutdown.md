---
tags: [NET, console-app]
share: true
---
# Безопасное завершение работы консольного приложения
Предположим, у нас есть некое консольное приложение, которое выполняет некое достаточно долгосрочное задание. Ну или открытый сокет держит, мало ли. Еще и в лог что-то пишет. Тогда при закрытии приложения неплохо перед остановкой корректно закрыть все соединения, зафиксировать факт останова в логе, и только уж потом завершить работу.
Для этого подпишемся на соответствующие события, чтобы обнаружить факт получения приложением сигналов SIGINT (обычно при помощи `Ctrl+C` с клавиатуры) и SIGTERM (такой сигнал посылает, например, Docker при остановке контейнера). Вот простейшее приложение:
```csharp
var tcs = new TaskCompletionSource();  
var sigintReceived = false;  
  
Console.WriteLine("Waiting for SIGINT/SIGTERM");  
  
Console.CancelKeyPress += (sender, eventArgs) =>  
{  
   eventArgs.Cancel = true;  
  
   Console.WriteLine("Received SIGINT (Ctrl+C)");  
  
   tcs.SetResult();  
   sigintReceived = true;  
};  
  
AppDomain.CurrentDomain.ProcessExit += (_, _) =>  
{  
   if(!sigintReceived)  
   {  
      Console.WriteLine("Received SIGTERM");  
      tcs.SetResult();  
   }  
   else  
   {  
      Console.WriteLine("Received SIGTERM, ignoring it because already processed SIGINT");  
   }};  
  
await tcs.Task;  
  
Console.WriteLine("Good bye");
```
Как видим, для передачи информации о необходимости останова использован [[task-completion-source|TaskCompletionSource]]. 
Также необходимо в обработчике сигнала SIGTERM проверять — а не получали ли мы SIGINT? Это нужно потому, что, например в Windows после SIGINT отправляется и SIGTERM. Вот что мы получим, запустив наше приложение:

```console
Waiting for SIGINT/SIGTERM
Received SIGINT (Ctrl+C)
Good bye
Received SIGTERM, ignoring it because already processed SIGINT 
```

При этом учтём, что при использовании [[generic-host|Generic Host]] нет нужды заботиться об обработке SIGINT и SIGTERM - хост делает это за нас.

## Ссылки
https://medium.com/@rainer_8955/gracefully-shutdown-c-apps-2e9711215f6d
