---
share: true
tags:
 - NET/SignalR
---
# Настройки протокола JSON
К вызову метода `AddSignalR()` добавим метод `AddJsonProtocol()` с настройками:
```csharp
builder.Services.AddSignalR(/*см. раздел о верхнеуровневых настройках*/)
	.AddJsonProtocol(options => 
	{
		options.PayloadSerializerOptions.PropertyNamingPolicy = null;
		options.PayloadSerializerOptions.Encoder = null;
		options.PayloadSerializerOptions.InclideFields = false;
		options.PayloadSerializerOptions.IgnoreReadOnlyFields = false;
		options.PayloadSerializerOptions.IgnoreReadOnlyProperties = false;
		options.PayloadSerializerOptions.MaxDepth = 0;
		options.PayloadSerializerOptions.NumberHandling = JsonNumberHandling.Strict;
		options.PayloadSerializerOptions.DictionaryKeyPolicy = null;
		options.PayloadSerializerOptions.DefaultIgnoreCondition = JsonIgnoreCondition.Never;
		options.PayloadSerializerOptions.PropertyNameCaseInsensitive = false;
		options.PayloadSerializerOptions.DefaultBufferSize = 32_768;
		options.PayloadSerializerOptions.ReadCommentHandling = System.Text.Json.JsonCommentHandling.Skip;
		options.PayloadSerializerOptions.ReferenceHandler = null;
		options.PayloadSerializerOptions.UnknownTypeHandling = JsonUnknownTypeHandling.JsonElement;
		options.PayloadSerializerOptions.WriteIndented = true;

		Console
		.WriteLine($"Number of default JSON converters: {options.PayloadSerializerOptions.Converters.Count}");
	});
```

Несмотря на то, что протокол JSON включён по умолчанию, использование `AddJsonProtocol` позволяет настроить его. Описание настроек есть в [документации](https://learn.microsoft.com/en-us/dotnet/api/system.text.json.jsonserializeroptions?view=net-6.0).