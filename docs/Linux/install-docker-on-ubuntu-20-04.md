---
share: true
tags: [docker, Linux]
---
# Установка Docker на Ubuntu 20.04
Обновим пакеты
```bash
sudo apt update
```
Установим пакеты для использования apt через HTTPS
```bash
sudo apt install apt-transport-https ca-certificates curl software-properties-common
```
Добавим ключ GPG от репозитория docker
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
добавим репозиторий docker в источники
```bash
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
```
Снова обновим пакеты
```bash
sudo apt update
```
Переключим репозиторий по умолчанию на docker
```bash
apt-cache policy docker-ce
```
Теперь установим docker
```bash
sudo apt install docker-ce
```
Проверим, что служба запущена:
```bash
sudo systemctl status docker
```

Если всё хорошо, можно [[docker-no-sudo|настроить использование без sudo]].
## Ссылки
https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04-ru
