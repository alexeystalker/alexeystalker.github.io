---
share: true
tags:
 - NET/configuration
 - NET/environment
---
# Задаем окружение размещения
Рассмотрим несколько способов задать окружение размещения.
Прежде всего - если переменная окружения `ASPNETCORE_ENVIRONMENT` отсутствует, по умолчанию используется промышленное окружение (`Production`).

Можно использовать файл launchSettings.json, добавляемый в папку Properties по умолчанию. Вот содержимое типичного файла launchSettings.json:
```json
{
	"iisSettings": {
		"windowsAuthentication": false,
		"anonymousAuthentication": true,
		"iisExpress": {
			"applicationUrl": "http://localhost:53846",
			"sslPort": 44399
		}
	},
	"profiles": {
		"IIS Express": {
			"commandName": "IISExpress",
			"launchBrowser": true,
			"environmentVariables": {
				"ASPNETCORE_ENVIRONMENT": "Development"
			}
		},
		"StoreViewerApplication": {
			"commandName": "Project",
			"launchBrowser": true,
			"environmentVariables": {
				"ASPNETCORE_ENVIRONMENT": "Development"
			},
			"applicationUrl": "https://localhost:5001;http://localhost:5000"
		}
	}
}
```
переменная окружения задана в разделе `environmentVariables`.
еще один способ — аргумент командной строки. Например
```sh
dotnet run --no-launch-profile --environment Staging
```
Важно, что при наличии файла launchSettings.json необходимо передавать `--no-launch-profile`, так как значения в файле имеют приоритет.