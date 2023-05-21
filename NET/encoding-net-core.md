---
tags: [NET/encoding,NET/Core]
share: true
---
# Encoding в NetCore
Всё время забываю: приложения на .NET Core на старте не готовы работать с разными кодировками (в особенности с Windows-related, типа cp1251). Чтобы всё взлетело, нужно в момент старта приложения зарегистрировать провайдер кодировок (стандартный или свой).
```csharp
Encoding.RegisterProvider(CodePagesEncodingProvider.Instance);
```
## Ссылки
https://stackoverflow.com/questions/3967716/how-to-find-encoding-for-1251-codepage
