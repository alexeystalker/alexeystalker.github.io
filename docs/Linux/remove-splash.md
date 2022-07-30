---
share: true
tags: [Linux/Mint, Linux/grub]
---
# Как убрать splash в Linux Mint
Возможно, работает во всех дебиано-подобных
```bash
sudo nano /etc/default/grub
```
Меняем строчку (впрочем, я сохранил исходную с закомментированием)
```
GRUB_CMDLINE_LINUX_DEFAULT="nosplash"
```
Дальше обязательно
```bash
sudo update-grub
```
## Ссылки
https://forums.linuxmint.com/viewtopic.php?t=70311

