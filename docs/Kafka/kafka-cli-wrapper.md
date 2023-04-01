---
share: true
tags:
 - Kafka
 - Linux
 - console
---
# Обёртка над консольными командами Kafka
Консольные команды Kafka размазаны по нескольким файлам `.sh`. Кроме того, все консольные команды Kafka требуют указания параметра `--bootstrap-server`, в котором должны быть указаны URL всех серверов, для которых выполняется команда. Это доставляет некоторое неудобство. Вот какое решение я подсмотрел.
Нужно создать *функцию* (назовём её kfk), куда передавать нужные параметры и нужный суффикс `.sh` файла.
```bash
kfk ()
{
    argv=("$@");
    cmd="kafka-$1.sh";
    for ((i=1; i<$#; i++ ))
    do
        arguments+=" ${argv[i]}";
    done;
    cmd+="$arguments --bootstrap-server kafka.srv1.ru:9092,kafka.srv2.ru:9092,kafka.srv3.ru:9092";
    echo "Execute 'command $cmd'";
    $cmd;
    unset cmd;
    unset arguments
}
```
определение этой функции можно добавить (в виде исполняемого `.sh` файла) в, например, `/etc/profile.d`. Или в локальном `.bashrc`.
После этого команду
```bash
kafka-topics.sh --bootstrap-server localhost:9092 --topic first_topic --create --partitions 3 --replication-factor 1
```
можно будет выполнить так:
```bash
kfk topics --topic first_topic --create --partitions 3 --replication-factor 1
```

## Ссылки
https://www.conduktor.io/kafka/kafka-cli-tutorial/
