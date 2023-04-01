---
share: true
tags:
 - Kafka
 - console
---
# Меняем настройки для конкретного топика
Сменить настройку для конкретного топика можно с помощью `kafka-configs`, указав `--entity-type` `topics`, имя топика, ключевое слово `--alter` и  значение настройки через `--add-config`. Например, выставим для топика `example-topic` настройку `retention.ms` на три часа  (пример ниже с использованием [[kafka-cli-wrapper|обёртки]]) 
```bash
kfk configs --entity-type topics --entity-name example-topic --alter --add-config retention.ms=10800000
```

## Ссылки
https://kafka.apache.org/documentation/#topicconfigs