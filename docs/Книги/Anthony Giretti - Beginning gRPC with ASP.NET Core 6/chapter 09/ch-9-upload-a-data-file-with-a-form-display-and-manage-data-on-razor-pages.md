---
share: true
tags:
 - NET/Razor
---
# Загружаем файл с данными, отображаем и управляем данными при помощи Razor Pages
> [!Note] Примечание
> Целью данной книги является демонстрация возможностей использования gRPC для разработки приложения с использованием ASP.NET Core и .NET 6. Поэтому мы опустим некоторые объяснения использования ASP.NET Core Razor Pages. Если вы хотите изучить вопрос подробнее, прочтите документацию Microsoft [по этой ссылке](https://learn.microsoft.com/en-us/aspnet/core/razor-pages/?view=aspnetcore-6.0)[^1]

[^1]: Или [[book-asp-net-core-in-action|книгу Эндрю Лока]]

Начнем со страницы **Index.cshtml**, на которую добавим форму для загрузки файла и HTML-таблицу со списком стран.
Вот Razor-разметка для страницы:
```razor
@page
@model IndexModel
@{
    ViewData["Title"] = "Country Wiki main page";
}

<div class="text-center">
    <h1 class="display-5">Country Wiki main page</h1>
</div>

<form method="post" enctype="multipart/form-data">
    <div class="container mb-5 mt-5">
        <div>
            Upload countries (JSON only):
            <input type="file" asp-for="Upload" />
        </div>
        <div><input type="submit" value="Upload" asp-page-handler="upload"/></div>
        <div class="text-danger">@Model.UploadErrorMessage</div>
        
        @if (Model.GlobalOptions.ProcessingUpload)
        {
            <div class="text-center text-danger"><h2>A file upload is in progress...</h2></div>
        }
    </div>
    
    <table class="table">
        <thead>
            <tr>
                <th>ID</th>
                <th>Name</th>
                <th>Description</th>
                <th>Capital City</th>
                <th>Anthem</th>
                <th>Spoken languages</th>
                <th>Flag</th>
                <th>Edit</th>
                <th>Delete</th>
            </tr>
        </thead>
        <tbody>
            @foreach (var country in Model.Countries)
            {
                <tr>
                    <td>@country.Id</td>
                    <td>@country.Name</td>
                    <td>@country.Description</td>
                    <td>@country.CapitalCity</td>
                    <td>@country.Anthem</td>
                    <td>@string.Join(", ", country.Languages)</td>
                    <td><img src="@country.FlagUri" alt="@country.Name" height="25" width="45"/></td>
                    <td><a asp-page="./Edit" asp-route-id="@country.Id">Edit</a></td>
                    <td><input type="submit" asp-page-handler="delete" asp-route-id="@country.Id" value="Delete" /></td>
                </tr>
            }
        </tbody>
    </table>
</form>
```
Обратите внимание, что поле `GlobalOptions.ProcessingUpload` используется для определения, нужно ли отображать сообщение о том, обрабатывается ли загруженный файл.
Вот код `IndexModel` с [[page-handler|обработчиками страницы]].
```csharp
namespace CountryWiki.Web.Pages;
public class IndexModel : PageModel
{
    private readonly ICountryServices _countryServices;
    private readonly ICountryFileUploadValidatorService _countryFileUploadValidatorService;
    private readonly ISyncCountriesChannel _syncCountriesChannel;

    public readonly GlobalOptions GlobalOptions;
    public IEnumerable<CountryModel> Countries { get; set; } = new List<CountryModel>();
    public string UploadErrorMessage { get; set; } = string.Empty;

    [BindProperty]
    public IFormFile? Upload { get; set; }  

    public IndexModel(
        ICountryServices countryServices,
        ICountryFileUploadValidatorService countryFileUploadValidatorService,
        ISyncCountriesChannel syncCountriesChannel,
        GlobalOptions globalOptions)
    {
        _countryServices = countryServices;
        _countryFileUploadValidatorService = countryFileUploadValidatorService;
        _syncCountriesChannel = syncCountriesChannel;
        GlobalOptions = globalOptions;
    }

    public async Task OnGetAsync()
    {
        Countries = await _countryServices.GetAllAsync();
    }

    public async Task<IActionResult> OnPostUploadAsync(CancellationToken cancellationToken)
    {
        if (Upload == null)
        {
            return await HandleFileValidation("File is missing");
        }
        var uploadedFile = new CountryUploadedFileModel
        {
            FileName = Upload.FileName,
            ContentType = Upload.ContentType
        };
        if (!_countryFileUploadValidatorService.ValidateFile(uploadedFile))
        {
            return await HandleFileValidation("Only JSON files are allowed");
        }
        var parsedCountries = await _countryFileUploadValidatorService
            .ParseFile(Upload.OpenReadStream());
        var createCountryModels = parsedCountries?.ToList();
        if (createCountryModels == null || !createCountryModels.Any())
        {
            return await HandleFileValidation("Cannot parse the file or the file is empty");
        }
        await _syncCountriesChannel.SyncAsync(createCountryModels, cancellationToken);
        return RedirectToPage("./Index");
    }

    public async Task<IActionResult> OnPostDeleteAsync(int id)
    {
        await _countryServices.DeleteAsync(id);
        return RedirectToPage("./Index");
    }

    private async Task<PageResult> HandleFileValidation(string errorMessage)
    {
        UploadErrorMessage = errorMessage;
        Countries = await _countryServices.GetAllAsync();
        return Page();
    }
}
```
Здесь реализованы следующие обработчики:
- `OnGetAsync` для загрузки страницы. Загружает страны при помощи `ICountryServices.GetAllAsync()`[^2];
- `OnPostUploadAsync` валидирует файл при помощи `ICountryFileUploadValidatorService`, после чего отправляет файл в `ISyncCountriesChannel`[^3]. Если всё проходит удачно, возвращается редирект[^4] на страницу;
- `OnPostDeleteAsync` удаляет страну при помощи `ICountryServices.DeleteAsync()`.

[^2]: Реализация `ICountryServices` и `ICountryFileUploadValidatorService` описана [[ch-9-write-the-business-logic-into-the-countryservice-bll-layer|здесь]]
[^3]: Реализация канала описана [[ch-9-create-a-background-task-for-handling-uploaded-file-data-and-create-a-channel-to-store-data|здесь]]
[^4]: Это пример [[prg-pattern|шаблона “POST-REDIRECT-GET”]]

Теперь подготовим файл с данными, например, такой:
```json
[
	{
		"name": "Canada",
		"description": "Maple leaf country",
		"capitalCity": "Ottawa",
		"anthem": "O Canada!",
		"flagUri": "https://anthonygiretti.blob.core.windows.net/countryflags/ca.png",
		"languages": [1, 2]
	},
	{
		"name": "USA",
		"description": "Uncle Sam country",
		"capitalCity": "Washington",
		"anthem": "The Star-Spangled Banner",
		"flagUri": "https://anthonygiretti.blob.core.windows.net/countryflags/us.png",
		"languages": [2, 3]
	},
	{
		"name": "United Kingdom",
		"description": "Sovereign country of North-western Europe",
		"capitalCity": "London",
		"anthem": "God save the Quuen",
		"flagUri": "https://anthonygiretti.blob.core.windows.net/countryflags/gb.png",
		"languages": [1]
	},
	{
		"name": "France",
		"description": "Human rights country",
		"capitalCity": "Paris",
		"anthem": "La marseillaise",
		"flagUri": "https://anthonygiretti.blob.core.windows.net/countryflags/fr.png",
		"languages": [2]
	},
	{
		"name": "Mexico",
		"description": "Cradle of civilization country",
		"capitalCity": "Mexico City",
		"anthem": "Himno Nacional Mexicano",
		"flagUri": "https://anthonygiretti.blob.core.windows.net/countryflags/mx.png",
		"languages": [3]
	}
]
```
Теперь, если мы загрузим файл и отправим его на обработку, нам покажут сообщение о том, что файл обрабатывается. Если после этого перезагрузить страницу, мы увидим загруженные данные.

Перейдём к странице редактирования. Вот её разметка:
```razor
@page
@model EditModel
@{
}
<h1>Edit Country - @Model.CountryName</h1>
<form method="post">
    <div asp-validation-summary="ModelOnly"></div>
    <input asp-for="CountryToUpdate.Id" type="hidden" value="@Model.CountryToUpdate.Id" />
    <div>
        <label asp-for="CountryToUpdate.Description"></label>
        <div>
            <textarea rows="4" cols="50" name="@Html.NameFor(m => m.CountryToUpdate.Description)">@Model.CountryToUpdate.Description</textarea>
        </div>
        <div><span asp-validation-for="CountryToUpdate.Description" class="text-danger"></span></div>
    </div>
    <div>
        <input type="submit" value="Save" />
    </div>
</form>
```
Вот модель
```csharp
namespace CountryWiki.Web.Pages;
public class EditModel : PageModel
{
    private readonly ICountryServices _countryServices;
    public string CountryName { get; set; }

    [BindProperty]
    public UpdateCountry CountryToUpdate { get; set; }

    public EditModel(ICountryServices countryServices)
    {
        _countryServices = countryServices;
    }
    public async Task OnGetAsync(int id)
    {
        await RetrieveCountry(id);
    }

    public async Task<IActionResult> OnPostAsync()
    {
        if (!ModelState.IsValid)
        {
            await RetrieveCountry(CountryToUpdate.Id);
            return Page();
        }

        await _countryServices.UpdateAsync(new UpdateCountryModel
        {
            Id = CountryToUpdate.Id,
            Description = CountryToUpdate.Description
        });
        return RedirectToPage("./Index");
    }

    private async Task RetrieveCountry(int id)
    {
        var country = await _countryServices.GetAsync(id);
        CountryName = country.Name;
        CountryToUpdate = new UpdateCountry
        {
            Id = country.Id,
            Description = country.Description
        };
    }
}
```
Здесь использована DTO `UpdateCountry`, вот код:
```csharp
namespace CountryWiki.Web.Models;

public class UpdateCountry
{
    public int Id { get; set; }

    [Required, StringLength(200, MinimumLength = 10)]
    public string Description { get; set; }
}
```
Мы пометили с помощью атрибута, что свойство `Description` для прохождения валидации должно быть заполнено и иметь длину минимум в 10, а максимум в 200 символов.