---
share: true
---
# Создаём и разделяем на слои приложение ASP.NET Core Razor
После того, как завершена работа над сервисом gRPC, мы создадим веб-приложение, позволяющее загрузить файл с начальными данными о странах, и затем отображать загруженные данные и изменять их.
#### Создадим каркас проекта
Прежде всего доработаем структуру нашего солюшена: добавим папки **gRPC** и **Web** и перенесем наши проекты **CountryService.\*** в папку **gRPC**. В папке **Web** создадим четыре проекта с префиксом **CountryWiki.\***: приложение ASP.NET Razor Pages под названием **CountryWiki.Web** и три библиотеки классов: **CountryWiki.Domain**, **CountryWiki.DAL** и **CountryWiki.BLL**.
#### [[ch-9-define-contracts-and-domain-objects|Определяем контракты и доменные объекты]]
#### [[ch-9-implement-the-data-access-layer-with-the-grpc-client|Реализуем слой доступа к данным при помощи клиента gRPC]]
#### [[ch-9-implement-the-business-logic-layer|Реализуем слой бизнес-логики]]
#### [[ch-9-configure-the-asp-net-core-razor-pages-application|Настраиваем приложение Razor Pages]]