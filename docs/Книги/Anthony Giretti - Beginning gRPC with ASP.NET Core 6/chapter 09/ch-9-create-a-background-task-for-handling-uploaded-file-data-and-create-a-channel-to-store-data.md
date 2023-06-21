---
share: true
tags:
 - NET/multithreading
 - NET/background-task
---
# Создадим фоновую задачу для обработки загруженного файла и канал для хранения данных
ASP.NET Core позволяет запускать фоновые задачи в виде *hosted services*, которые будут выполняться до тех пор, пока запущено основное приложение. Это удобно для реализации долговременных операций. В нашем сценарии данные о странах из загруженного файла будут синхронизироваться с удалённым источником данных при помощи gRPC. Это может занять довольно долгое время, и поэтому мы должны дать пользователю возможность использовать приложение сразу после загрузки и валидации файла, не блокируя его до окончания синхронизации.
Вот реализация класса `SyncCountriesChannel` и его интерфейса `ISyncCountriesChannel`. Он понадобится нам только в приложении Razor Pages, поэтому создадим его в соответствующем проекте, при этом интерфейс и реализацию разместим в одном файле.
```csharp
namespace CountryWiki.Web.Channels;

public interface ISyncCountriesChannel
{
    IAsyncEnumerable<IEnumerable<CreateCountryModel>> ReadAllAsync(CancellationToken cancellationToken);
    Task<bool> SyncAsync(
        IEnumerable<CreateCountryModel> countriesToCreate, CancellationToken cancellationToken);
}
public class SyncCountriesChannel : ISyncCountriesChannel
{
    private readonly ILogger<SyncCountriesChannel> _logger;
    private readonly Channel<IEnumerable<CreateCountryModel>> _channel;

    public SyncCountriesChannel(ILogger<SyncCountriesChannel> logger)
    {
        var options = new UnboundedChannelOptions
        {
            SingleWriter = false,
            SingleReader = true
        };
        _channel = Channel.CreateUnbounded<IEnumerable<CreateCountryModel>>(options);
        _logger = logger;
    }

    public async Task<bool> SyncAsync(IEnumerable<CreateCountryModel> countriesToCreate, CancellationToken cancellationToken)
    {
        while (await _channel.Writer.WaitToWriteAsync(cancellationToken)
               && !cancellationToken.IsCancellationRequested)
        {
            if (_channel.Writer.TryWrite(countriesToCreate))
            {
                _logger.LogDebug("Sending parsed countries to the background task");
                return true;
            }
        }
        return false;
    }

    public IAsyncEnumerable<IEnumerable<CreateCountryModel>> ReadAllAsync(
        CancellationToken cancellationToken) => _channel.Reader.ReadAllAsync(cancellationToken);
}
```
Этот класс создаёт [[system-threading-channels-channel|канал]] (из пространства имён `System.Threading.Channels`), который позволяет безопасно передавать данные от потока-создателя (producer) к потоку-потребителю (consumer). В *канал* передаётся один объект за раз, и один же объект за раз читается (с изъятием из *канала*). Здесь мы использовали “неограниченный” (unbounded) канал, то есть канал без задания лимита на число содержащихся элементов. Это можно сделать, так как предполагается, что файлы будут загружаться достаточно редко. Писателей может быть несколько (мы указали это в объекте `UnboundChannelOptions`), поэтому мы проверяем возможность записи в канал при помощи метода `WaitToWriteAsync`. Как видно, мы предполагаем использовать одного читателя — фоновую задачу.

---
Далее создадим глобальную переменную для хранения статуса обработки загруженных файлов. Сначала создадим класс `GlobalOptions`.
```csharp
namespace CountryWiki.Web.Options;

public class GlobalOptions
{
    public bool ProcessingUpload { get; set; }
}
```
Для того, чтобы сделать объект этого класса *глобальным*, нужно всего лишь зарегистрировать его в [[di-container|контейнере зависимостей]] с [[dependency-injection-pattern|жизненным циклом Singleton]].

---
Теперь реализуем нашу фоновую задачу. Для этого нужно унаследоваться от абстрактного класса `BackgroundService` и реализовать метод `ExecuteAsync()`[^1]
```csharp
namespace CountryWiki.Web.Background;

public class SyncUploadedCountriesBackgroundService : BackgroundService
{
    private readonly ILogger<SyncUploadedCountriesBackgroundService> _logger;
    private readonly ISyncCountriesChannel _syncCountriesChannel;
    private readonly IServiceProvider _serviceProvider;
    private readonly GlobalOptions _globalOptions;

    public SyncUploadedCountriesBackgroundService(
        ILogger<SyncUploadedCountriesBackgroundService> logger,
        ISyncCountriesChannel syncCountriesChannel,
        IServiceProvider serviceProvider,
        GlobalOptions globalOptions)
    {
        _logger = logger;
        _syncCountriesChannel = syncCountriesChannel;
        _serviceProvider = serviceProvider;
        _globalOptions = globalOptions;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        await foreach (var uploadedCountries in _syncCountriesChannel.ReadAllAsync(stoppingToken))
        {
            try
            {
                _logger.LogInformation("Received uploaded countries from the channel for sync");
                using var scope = _serviceProvider.CreateScope();
                var countryServices = scope.ServiceProvider.GetRequiredService<ICountryServices>();
                try
                {
                    // Синхронизируем
                    _globalOptions.ProcessingUpload = true;
                    await countryServices.CreateAsync(uploadedCountries);
                }
                catch (RpcException e)
                {
                    var correlationId = e.Trailers.GetValue("correlationId");
                    _logger.LogError(
                        e,
                        "Background synchronization has failed. CorrelationId {correlationId}",
                        correlationId);
                }
                finally
                {
                    _globalOptions.ProcessingUpload = false;
                }
            }
            catch (Exception e)
            {
                _logger.LogError(e, "Unable to manage uploaded countries");
            }
            finally
            {
                _globalOptions.ProcessingUpload = false;
            }
        }
    }
}
```
Так как объект фоновой задачи будет создан с жизненным циклом Singleton, также, как и его зависимости, нам нужно создавать новый scope всякий раз, когда мы хотим использовать объект `ICountryServices`, и поэтому нам нужно внедрить ссылку на `IServiceProvider` для получения scope.

[^1]: Также можно прочесть о фоновых задачах у Э. Лока [[ch-22-building-background-task-and-services|здесь]].