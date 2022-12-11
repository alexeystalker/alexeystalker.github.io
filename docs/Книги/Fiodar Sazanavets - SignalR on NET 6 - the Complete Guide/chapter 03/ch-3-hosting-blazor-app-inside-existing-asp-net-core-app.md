---
share: true
tags:
 - NET/ASPNETCore
 - NET/BlazorWasm
---
# Размещаем приложение Blazor внутри приложения ASP.NET Core
Наше [[ch-3-setting-up-blazor-wasm-signalr-client-components|существующее приложение]] Blazor по умолчанию сконфигурировано как самостоятельное приложение, и нам нужно внести ещё несколько изменений, чтобы разместить его внутри другого приложения.  Для этого сперва удалим следующую строчку в файле **BlazorClient/Program.cs**:
```csharp
builder.RootComponents.Add<App>("#app");
```
Далее, добавим проект BlazorClient в качестве зависимости к проекту SignalRServer.
Затем нам нужно добавить специальный NuGet-пакет, который позволит приложению SignalRServer выступать сервером для размещения приложения Blazor WebAssembly:
```bash
dotnet add package Microsoft.AspNetCore.Components.WebAssembly.Server
```
Далее, добавим страницу, которая будет отображать наш компонент Blazor. Откроем файл **Controllers/HomeController.cs** нашего приложения и добавим метод
```csharp
public IActionResult WebAssemblyClient()
{
	return View();
}
```
Затем добавим соответствующее отображение. Создадим файл **Views/Home/WebAssemblyClient.cshtml**, в котором будет следующее:
```razor
@{
	ViewData["Title"] = "Home Page";
}
@using BlazorClient.Pages;
<component type="typeof(Client)" render-mode="WebAssemblyPrerendered" />
<script src="_framework/blazor.webassembly.js"></script>
```
Здесь мы загрузили компонент `Client` из сборки BlazorClient, а также добавили загрузку js-файла, которых позволит компоненту WebAssembly взаимодействовать с контролами на нашей странице. Этот файл поставляется в установленном нами NuGet-пакете. Режим сборки WebAssemblyPrerendered указывает платформе на то, что код будет собран как WebAssembly. Если использование WebAssembly невозможно (например, страница открыта в старом браузере, или браузере с жёсткими ограничениями), будет сгенерирован JavaScript код (аналогично тому, как действует Blazor Server).
Далее добавим ссылку на только что созданную страницу. Откроем файл **Views/Shared/\_Layout.cshtml** и найдём элемент, отвечающий за навигационную панель. Это элемент `ul` с классом `navbar-nav`. Добавим в него следующее:
```html
<li class="nav-item">
	<a class="nav-link text-dark" asp-area="" asp-controller="Home" asp-action="WebAssemblyClient">WebAssembly</a>
</li>
```
И, наконец, обеспечим доступ к необходимым файлам для промежуточного ПО нашего приложения. Добавим следующую строку в файл **Program.cs** перед вызовом `app.Run()`:
```csharp
app.UseBlazorFrameworkFiles();
```