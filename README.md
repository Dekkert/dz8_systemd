#   Загрузка системы 

Задание:

```text
 1.   Написать service, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова (файл лога и ключевое слово должны задаваться в /etc/sysconfig).
 2.   Из репозитория epel установить spawn-fcgi и переписать init-скрипт на unit-файл (имя service должно называться так же: spawn-fcgi).
 3.   Дополнить unit-файл httpd (он же apache) возможностью запустить несколько инстансов сервера с разными конфигурационными файлами.
```

Выполнение:
1.	Написать service, который будет раз в 30 секунд мониторить лог на предмет наличия ключевого слова (файл лога и ключевое слово должны задаваться в /etc/sysconfig).

Создаём файл с конфигурацией для сервиса в директории /etc/sysconfig - из неё сервис будет брать необходимые переменные.

cd /etc/sysconfig/
vi watchlog

Содержимое файла

```bash
# Configuration file for my watchlog service
# Place it to /etc/sysconfig
# File and word in that file that we will be monit
keyword="ALERT"
logfile="/var/log/watchlog.log"
```

Затем создаем /var/log/watchlog.log и пишем туда строки на своё усмотрение, плюс ключевое слово ‘ALERT’

```bash
cd /var/log/
vi watchlog
```

Создадим скрипт watchlog.sh :

```bash
cd /opt/
vi watchlog
```

Содержимое файла (команда logger отправляет лог в системный журнал):

```bash
#!/bin/bash

if [[ ! "$logfile" ]]; then
    echo "Please provide filename" >&2
    exit 1
fi

if [[ ! "$keyword" ]]; then
    echo "Please provide key word" >&2
fi

echo "Searching ${keyword} in ${logfile}"

grep "$keyword" "$logfile"
```

Создадим юнит для сервиса:

```bash
cd /etc/systemd/system
vi watchlog.service
```

Содержимое файла watchlog.service :

```bash
[Unit]
After=network.target

[Service]
EnvironmentFile=/etc/sysconfig/watchlog
WorkingDirectory=/home/vagrant
ExecStart=/bin/bash '/opt/watchlog.sh'
Type=oneshot

[Install]
WantedBy=multi-user.target
```

Создадим юнит для таймера:

vi watchlog.timer

Содержимое файла watchlog.timer:

```bash
[Unit]
Description=Run every 30 seconds

[Timer]
OnBootSec=1m
OnUnitActiveSec=30s
Unit=watchlog.service

[Install]
WantedBy=timers.target
```

Затем достаточно толþко стартанутþ timer:

```bash
systemctl start watchlog.timer
```

И убедитþсā в резулþтате:
```bash
tail -f /var/log/messages
```

Ссылка на репозиторий с Vagrnatfile и скриптами:


2.	Из репозитория epel установить spawn-fcgi и переписать init-скрипт на unit-файл (имя service должно называться так же: spawn-fcgi).



