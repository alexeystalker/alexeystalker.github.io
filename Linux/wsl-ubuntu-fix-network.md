---
share: true
tags:
  - Linux
  - WSL
  - network 
---
# Чиним сеть в WSL Ubuntu 20.04
У меня внезапно отвалилась сеть в WSL (Windows Subsystem for Linux), в которой крутится Ubuntu 20.04. Не очень понятно почему это произошло. Но вот рецепт, который помог мне вернуть всё обратно.
Открываем (в самой Windows) консоль с правами администратора, затем вводим следующие команды:
```
netsh winsock reset 
netsh int ip reset all
netsh winhttp reset proxy
ipconfig /flushdns
```
После чего перезагружаемся.

## Ссылки
https://stackoverflow.com/a/63578387
[Коммент к issue на Github](https://github.com/microsoft/WSL/issues/3438#issuecomment-410518578)