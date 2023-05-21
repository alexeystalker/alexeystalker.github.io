---
share: true
tags:
 - gRPC/compression
---
# Добавим провайдер сжатия
Мы можем определить кастомный провайдер сжатия передаваемых данных.

Для примера, добавим провайдер, реализующий [алгоритм Brotli](https://ru.wikipedia.org/wiki/Brotli).
```csharp
using System.IO.Compression;
using Grpc.Net.Compression;

namespace CountryService.gRPC.Compression;

public class BrotliCompressionProvider : ICompressionProvider
{
    private readonly CompressionLevel? _compressionLevel;

    public BrotliCompressionProvider(CompressionLevel compressionLevel)
    {
        _compressionLevel = compressionLevel;
    }

    public BrotliCompressionProvider() { }

    public Stream CreateCompressionStream(Stream stream, CompressionLevel? compressionLevel)
    {
        var currentCompressionLevel = compressionLevel ?? _compressionLevel ?? CompressionLevel.Fastest;

        return new BrotliStream(stream, currentCompressionLevel, true);
    }

    public Stream CreateDecompressionStream(Stream stream)
    {
        return new BrotliStream(stream, CompressionMode.Decompress);
    }

    public string EncodingName => "br"; //Это же должно быть указано в grpc-accept-encoding
}
```
Теперь добавляем опции к методу `AddGrpc`:
```csharp
builder.Services.AddGrpc(options =>
{
    options.MaxReceiveMessageSize = 6291456; // 6 Mb
    options.MaxSendMessageSize = 6291456; // 6 Mb
    options.CompressionProviders = new ICompressionProvider[] 
    {
		new BrotliCompressionProvider(CompressionLevel.Optimal)
	};
    options.ResponseCompressionAlgorithm = "br"; //задаём grpc-accept-encoding, соответствует указанному в провайдере
    options.ResponseCompressionLevel = CompressionLevel.Optimal;
});
```