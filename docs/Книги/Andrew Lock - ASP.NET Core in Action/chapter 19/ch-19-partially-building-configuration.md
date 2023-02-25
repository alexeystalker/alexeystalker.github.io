---
share: true
tags:
 - NET/configuration
---
# Частичное создание конфигурации для настройки дополнительных поставщиков
В некоторых случаях приложению необходимо использовать [[configuration-provider|поставщиков конфигурации]], которым для работы самим требуется конфигурация. Например, может быть поставщик, загружающий значения из базы данных или удалённого API, а также поставщик секретов, загружающий их из удалённого хранилища, такого, как [[hashicorp-vault|Hashicorp Vault]] или Azure Key Vault. Для работы им может потребоваться, соответственно, строка подключения к БД, URL-адрес удалённой службы или ключ расшифровки данных из хранилища секретов.

Решение состоит в том, чтобы использовать двухэтапный процесс для создания окончательного объекта `IConfiguration`. На первом этапе загружаются значения конфигурации, доступные локально, и создаётся переменная `IConfiguration`, затем этот объект используется для настройки других поставщиков, далее эти поставщики добавляются в конструктор конфигурации и создаётся окончательный объект `IConfiguration`.
![[Pasted image 20220915201139.png]]
В качестве примера рассмотрим, как создать временный объект `IConfiguration` из содержимого XML-файла, который содержит свойство `"SettingsFile"` с именем файла JSON. На втором этапе добавляем поставщика файлов JSON, используя полученное имя файла и поставщика переменной окружения.
```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
	Host.CreateDefaultBuilder(args)
		.ConfigureAppConfiguration((context, config) =>
		{
			config.Sources.Clear(); //Удаляем источники по умолчанию
			config.AddXmlFile("baseconfig.xml");
			
			var partialConfig = config.Build(); //Создали частичную конфигурацию
			var filename = partialConfig["SettingsFile"]; //Извлекаем значение
			config.AddJsonFile(filename) //Используем извлеченное значение
				.AddEnvironmentVariables();
		})
		.ConfigureWebHostDefaults(webBuilder => webBuilder.UseStartup());
	
```