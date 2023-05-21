---
share: true
tags: [docker, Linux/console]
---
# Docker без sudo
Как сделать так, чтобы команды докера можно было запускать без `sudo`.
Вкратце, для этого нужно добавить себя в группу `docker`.
Подробнее:
Попытаемся создать группу `docker` (создастся, если ее нет)
```bash
sudo groupadd docker
```
Затем добавляем текущего пользователя в группу
```bash
sudo gpasswd -a $USER docker  
```
Затем либо делаем logout/login, либо
```bash
newgrp docker
```
всё должно работать.
## Ссылки
https://itsecforu.ru/2018/04/12/%D0%BA%D0%B0%D0%BA-%D0%B8%D1%81%D0%BF%D0%BE%D0%BB%D1%8C%D0%B7%D0%BE%D0%B2%D0%B0%D1%82%D1%8C-docker-%D0%B1%D0%B5%D0%B7-sudo-%D0%BD%D0%B0-ubuntu/
