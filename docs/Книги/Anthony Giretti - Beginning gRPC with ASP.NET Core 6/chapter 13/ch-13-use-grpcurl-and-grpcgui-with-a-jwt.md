---
share: true
tags:
 - gRPC/gRPCurl
 - gRPC/gRPCui
---
# Используем gRPCurl и gRPCui с JWT
Настроенный в [[ch-13-configure-asp-net-core|предыдущем разделе]] способ аутентификации Bearer Token предполагает, что в HTTP-запросе передаётся заголовок вида `Authorization: Bearer {TOKEN}`, где вместо `{TOKEN}` подставляется JWT, закодированный в Base64.
## gRPCurl
Для инструмента командной строки [[ch-5-using-grpcurl|gRPCurl]] требуется указать дополнительный ключ `-H 'authorization: bearer {TOKEN}'`
## gRPCui
Для инструмента [[ch-5-using-grpcui|gRPCui]] нужно добавить строку в разделе **Request Metadata** вкладки **Request Form**. Добавляем строку, в поле **Name** указываем `authorization`, в поле **Value** — `bearer {TOKEN}`.
