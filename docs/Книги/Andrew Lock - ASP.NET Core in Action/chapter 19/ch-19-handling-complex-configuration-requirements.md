---
share: true
tags:
 - NET/configuration
---
# Работа с требованиями к сложной конфигурации
[[ch-11-configuring-application|Ранее]] мы рассмотрели систему конфигурации ASP.NET Core. В этой главе дополнительно разберем два более сложных, чем ранее, сценария.
Первый сценарий заключается в использовании [[configuration-provider|поставщика конфигурации]], для работы которого также требуются настройки.
Во втором сценарии необходимо настроить строго типизированный [[ch-11-introducing-ioptions-interface|объект]] `IOptions` со значениями, возвращаемыми из сервиса. Но сервис будет доступен только после того, как все объекты `IOptions` будут настроены.
#### [[ch-19-partially-building-configuration|Частичное создание конфигурации для настройки дополнительных поставщиков]]
#### [[ch-19-using-services-to-configure-ioptions-with-iconfigureoptions|Использование сервисов для настройки IOptions с помощью IConfigureOptions]]
