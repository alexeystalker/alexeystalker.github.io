---
share: true
tags:
 - NET/SignalR
 - NET/BlazorWasm
---
# Настраиваем клиент для Blazor WebAssembly
Blazor WebAssembly - это реализация технологии WebAssembly на .NET.
Важно учесть, что существует два разных вида Blazor - Blazor WebAssembly и Blazor Server. Первый вид позволяет скомпилировать код в WebAssembly и выполнять его в браузере (его мы и рассмотрим далее), тогда как второй предназначен для генерации JavaScript в процессе сборки приложения.
#### Создадим проект
Прежде всего создадим приложение Blazor WebAssembly и добавим его к решению. Для этого выполним в консоли, в папке нашего решения, команды
```bash
dotnet new blazorwasm -o BlazorClient
dotnet sln add BlazorClient/BlazorClient.csproj
```
#### [[ch-3-setting-up-blazor-wasm-signalr-client-components|Настроим компоненты SignalR]]
#### [[ch-3-hosting-blazor-app-inside-existing-asp-net-core-app|Размещаем приложение Blazor внутри приложения ASP.NET Core]]
#### Запускаем клиент Blazor
Теперь, если мы запустим наше приложение и перейдём на страницу WebAssembly, то увидим практически идентичную [[ch-3-setting-up-javascript-client|предыдущей]] страницу. Однако, если мы откроем обе страницы одновременно в разных вкладках, мы увидим, что посланное только с одной страницы сообщение отображается на обоих страницах. Это происходит потому, что мы отправляем сообщение, посланное одним клиентам, ВСЕМ подключенным клиентам.