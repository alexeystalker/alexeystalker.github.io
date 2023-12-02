---
share: true
tags:
 - docker
 - NAS/Synology
---
# Docker без sudo на Synology NAS

Если вам нужно выполнять команды `docker` в командной строке NAS Synology, а `sudo` звать не хочется, можно сделать следующее:
Сперва создать группу пользователей docker и добавить туда себя (ну, тут нужно звать `sudo`, никуда не деться):
```bash
sudo synogroup --add docker $USER
```
Далее, нужно переназначить владение сокетом на эту группу:
```bash
sudo chown root:docker /var/run/docker.sock
```
Всё, теперь всё должно работать. Возможно, нужно перезайти.

---
Если вам нужно вызывать команды `docker` под ubuntu, и не хочется каждый раз вызывать `sudo`, то вам [[./docker-no-sudo|сюда]].

## Ссылки
https://davejansen.com/manage-docker-without-needing-sudo-on-your-synology-nas/
