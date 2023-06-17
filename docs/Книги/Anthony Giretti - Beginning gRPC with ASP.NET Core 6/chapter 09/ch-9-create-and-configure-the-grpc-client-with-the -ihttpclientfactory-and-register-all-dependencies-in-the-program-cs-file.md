---
share: true
tags:
 - gRPC/client
 - gRPC/interceptors
 - gRPC/compression
---
# Создадим и сконфигурируем клиент gRPC при помощи IHttpClientFactory, а также зарегистрируем все зависимости в файле Program.cs
В [[ch-7-create-a-grpc-client|главе 7]] мы создавали клиент gRPC в консольном приложении. В этот раз мы будем создавать клиент при помощи `IHttpClientFactory`. Для этого есть специальный метод расширения `AddGrpcClient`.

> [!Note] От меня
> В книге автор делает очень, на мой взгляд, некорректное действие — импортирует файл Protobuf в проект CountryWiki.Web, чтобы потом проинициализировать клиента в **Program.cs** (что верно). Я считаю, что корректнее было бы инкапсулировать инициализацию клиента в проекте **CountryWiki.DAL** при помощи метода расширения. ПОЭТОМУ я отступлю от книги и сделаю метод расширения.

Сперва создадим метод расширения в проекте **CountryWiki.DAL**, в котором проинициализируем всё, что нужно.
```csharp
namespace CountryWiki.DAL;

public static class ServicesBuilderExtensions
{
    public static IServiceCollection AddCountryServiceClient(this IServiceCollection services,
        ILoggerFactory loggerFactory, string countryServiceUri)
    {
        services.AddGrpcClient<CountryServiceClient>(o =>
            {
                o.Address = new Uri(countryServiceUri);
            })
            .AddInterceptor(() => new TracerInterceptor(loggerFactory.CreateLogger<TracerInterceptor>()))
            .ConfigureChannel(o =>
            {
                o.CompressionProviders = new List<ICompressionProvider>
                {
                    new BrotliCompressionProvider()
                };
                o.MaxReceiveMessageSize = 6291456;
                o.MaxSendMessageSize = 6291456;
            });

        return services;
    }
}
```
Добавим `TracerInterceptor` [[ch-7-interceptors|отсюда]] и поставщик сжатия, аналогичный тому, что мы добавили в **CountryService.gRPC**.
Теперь можно использовать этот метод для добавления клиента в [[di-container|контейнер зависимостей]], наряду с другими сервисами. Вот какой у нас получится файл **Program.cs**:
```csharp
var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddRazorPages();
builder.Services.AddScoped<ICountryRepository, CountryRepository>();
builder.Services.AddScoped<ICountryServices, CountryServices>();
builder.Services.AddScoped<ICountryFileUploadValidatorService, 
    CountryFileUploadValidatorService>();
builder.Services.AddSingleton<ISyncCountriesChannel, SyncCountriesChannel>();
builder.Services.AddHostedService<SyncUploadedCountriesBackgroundService>();
builder.Services.AddSingleton(new GlobalOptions {ProcessingUpload = false});

var loggerFactory = LoggerFactory.Create(logging =>
{
    logging.AddConsole();
    logging.SetMinimumLevel(LogLevel.Trace);
});

builder.Services.AddCountryServiceClient(
    loggerFactory, 
    builder.Configuration.GetValue<string>("CountryServiceUri"));

//-- начиная с этой строки, весь код ниже взят напрямую из шаблона
var app = builder.Build();

// Configure the HTTP request pipeline.
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Error");
    // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();

app.UseRouting();

app.UseAuthorization();

app.MapRazorPages();

app.Run();
```
Обратим внимание, что Url сервиса gRPC мы поместили в файл настроек **appsettings.json**:
```json
{
//...
  "CountryServiceUri": "https://localhost:7117"
//...
}
```