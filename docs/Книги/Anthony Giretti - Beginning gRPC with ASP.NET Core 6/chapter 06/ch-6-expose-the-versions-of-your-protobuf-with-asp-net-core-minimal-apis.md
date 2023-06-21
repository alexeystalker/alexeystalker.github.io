---
share: true
tags:
 - versioning
 - gRPC/protobuf
 - NET/ASPNETCore
---
# Предоставляем версии файлов Protobuf при помощи ASP.NET Core Minimal API
Не всегда просто передавать файлы Protobuf клиентам, особенно, если файлов несколько, и у каждого существует несколько версий. Может быть затруднительно управлять версионированием, а также можно поставить клиента в неудобное положение, если, к примеру, изменить версию без того, чтобы уведомить клиента.
В этом разделе мы рассмотрим, как предоставить клиенту файлы Protobuf при помощи новой фичи ASP.NET Core — Minimal APIs.
Сделаем три эндпоинта, которые будут предоставлять файлы Protobuf:
- `/protos` — предоставляет версии всех сервисов в формате JSON;
- `/protos/v{version:int}/{protoName}` — позволит скачать файл Protobuf нужной версии;
- `/protos/v{version:int}/{protoName}/view` — показывает файл Protobuf в текстовом формате.
Прежде всего, напишем класс `ProtoService`:
```csharp
namespace CountryService.Web;

public class ProtoService
{
    private readonly string _baseDirectory;

    public ProtoService(IWebHostEnvironment webHostEnvironment)
    {
        _baseDirectory = webHostEnvironment.ContentRootPath;
    }

    public Dictionary<string, IEnumerable<string?>> GetAll() =>
        Directory
            .GetDirectories($"{_baseDirectory}/protos")
            .Select(x => new
            {
                version = x,
                protos = Directory.GetFiles(x).Select(Path.GetFileName)
            })
            .ToDictionary(o => Path.GetRelativePath("protos", o.version), o => o.protos);

    public string? Get(int version, string protoName)
    {
        var filePath = $"{_baseDirectory}/protos/v{version}/{protoName}";
        var exist = File.Exists(filePath);
        return exist ? filePath : null;
    }

    public async Task<string> ViewAsync(int version, string protoName)
    {
        var filePath = $"{_baseDirectory}/protos/v{version}/{protoName}";
        var exist = File.Exists(filePath);
        return exist ? await File.ReadAllTextAsync(filePath) : string.Empty;
    }
}
```
Зарегистрируем его в **Program.cs** как синглтон
```csharp
builder.Services.AddSingleton<ProtoService>(); 
```
Теперь добавим наши эндпоинты:
```csharp
// ... предыдущий код Program.cs ...
app.MapGet("/protos", (ProtoService protoService) => Results.Ok(protoService.GetAll()));
app.MapGet("/protos/v{version:int}/{protoName}", (ProtoService protoService, int version, string protoName) =>
{
    var filePath = protoService.Get(version, protoName);
    return filePath != null ? Results.File(filePath) : Results.NotFound();
});
app.MapGet("/protos/v{version:int}/{protoName}/view", async (ProtoService protoService, int version, string protoName) =>
{
    var text = await protoService.ViewAsync(version, protoName);
    return !string.IsNullOrEmpty(text) ? Results.Text(text) : Results.NotFound();
});

app.Run();
```

Теперь можно открыть в браузере соответствующие URL и посмотреть список файлов `proto` или получить нужный файл.