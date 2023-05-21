---
share: true
tags:
 - NET/ASPNETCore/Identity
---
# Настройка страницы в пользовательском интерфейсе ASP.NET Core Identity по умолчанию
В этом разделе рассмотрим, как использовать скаффолдинг для замены отдельных страниц в пользовательском интерфейсе Identity по умолчанию.
> [!tip] Определение
> _Скаффолдинг_ - это процесс генерации файлов в проекте, которые служат основой для настройки.

В качестве примера изменим страницу регистрации и удалим раздел дополнительной информации о внешних поставщиках. Опишем, как выполнить задачу в Visual Studio. Также есть возможность использовать интерфейс командной строки .NET[^1].
1. Добавляем пакеты Microsoft.VisualStudio.Web.CodeGeneration.Design и Microsoft.EntityFrameworkCore.Tools в проект - без них получим ошибку при запуске инструмента скаффолдинга;
2. Убеждаемся, что проект собирается;
3. ПКМ по проекту: **Add** > **New Scaffolded Item**;
4. В диалоговом окне выбираем **Identity**, нажимаем **Add**;
5. В диалоговом окне **Add Identity** выбираем **Account/Register** и выбираем `AppDbContext` в качестве класса DataContext и нажимаем **Add**.

VS собирает приложение, затем генерирует страницу Register.cshtml, поместив ее в Areas/Identity/Pages/Account. Также генерируется несколько вспомогательных файлов.
![[Pasted image 20220610200703.png]]
Во многих случаях не требуется менять обработчики страниц, а только представление. Этого можно добиться, удалив сгенерированнцю `PageModel` вместе с файлом Register.cshtml.cs,  и указав в файле представления исходную `PageModel` из пакета NuGet. Для этого нужно:
1. Обновить директиву `@model` в файле Register.cshtml:
	```razor
	@model Microsoft.AspNetCore.Identity.UI.V4.Pages.Account.Internal.RegisterModel
	```
1. Обновить файл Areas/Identity/Pages/\_ViewImports.cshtml:
	```razor
	@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
	```
1. Удалить файл Areas/Identity/Pages/IdentityHostingStartup.cs.
2. Удалить файл Areas/Identity/Pages/\_ValidationScriptsPartial.cshtml.
3. Удалить файл Areas/Identity/Pages/Account/Register.cshtml.cs.
4. Удалить файл Areas/Identity/Pages/Account/\_ViewImports.cshtml.


[^1]: Нужно установить необходимые инструменты и пакеты, как описано [здесь](https://docs.microsoft.com/en-us/aspnet/core/security/authentication/scaffold-identity?view=aspnetcore-5.0&tabs=netcore-cli), затем выполнить команду `dotnet aspnet-codegenerator identity -dc RecipeApplication.Data.AppDbContext --files "Account.Register"`.	