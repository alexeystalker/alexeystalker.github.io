---
share: true
tags: [Linux/ssh]
---
# Автозапуск ssh-agent
Если ssh-agent не запущен, как например у меня в WSL Debian, можно сделать следующее.
Добавим в `.profile` (или `.bash_profile`, если он есть)
следующий код:
```bash
SSH_ENV="$HOME/.ssh/agent-environment"

function start_agent {
    echo "Initialising new SSH agent..."
    /usr/bin/ssh-agent | sed 's/^echo/#echo/' > "${SSH_ENV}"
    echo succeeded
    chmod 600 "${SSH_ENV}"
    . "${SSH_ENV}" > /dev/null
    /usr/bin/ssh-add;
}

# Source SSH settings, if applicable

if [ -f "${SSH_ENV}" ]; then
    . "${SSH_ENV}" > /dev/null
    #ps ${SSH_AGENT_PID} doesn't work under cywgin
    ps -ef | grep ${SSH_AGENT_PID} | grep ssh-agent$ > /dev/null || {
        start_agent;
    }
else
    start_agent;
fi
```
Всё, теперь при запуске будет стартовать ssh-agent и выполняться команда `ssh-add ./.ssh/id_rsa`. Если ключик под паролем, будет просить ввести пароль.

## Ссылки
https://web.archive.org/web/20210506080335/https://mah.everybody.org/docs/ssh
[[web_archive_org_web_20210506080335_mah_everybody_org_docs_ssh.pdf]]