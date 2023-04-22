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
https://github.com/Dekkert/dz8_systemd/tree/master/httpd

2.	Из репозитория epel установить spawn-fcgi и переписать init-скрипт на unit-файл (имя service должно называться так же: spawn-fcgi).

Устанавливаем spawn-fcgi и необходимые для него пакеты:
```bash
yum install epel-release -y && yum install spawn-fcgi php php-cli mod_fcgid httpd -y
```
/etc/rc.d/init.d/spawn-fcgi - cам Init скрипт, которýй будем переписывать

Но перед этим необходимо раскомментировать строки с переменными в
/etc/sysconfig/spawn-fcgi

Он должен получиться следующего вида:

```bash
cat /etc/sysconfig/spawn-fcgi
# You must set some working options before the "spawn-fcgi" service will work.
# If SOCKET points to a file, then this file is cleaned up by the init script.
#
# See spawn-fcgi(1) for all possible options.
#
# Example :
SOCKET=/var/run/php-fcgi.sock
OPTIONS="-u apache -g apache -s $SOCKET -S -M 0600 -C 32 -F 1 -- /usr/bin/php-cgi"
```

А сам юнит файл будет примерно следующего вида:

```bash
сat /etc/systemd/system/spawn-fcgi.service
[Unit]
Description=Spawn-fcgi startup service by Otus
After=network.target

[Service]
Type=simple
PIDFile=/var/run/spawn-fcgi.pid
EnvironmentFile=/etc/sysconfig/spawn-fcgi
ExecStart=/usr/bin/spawn-fcgi -n $OPTIONS
KillMode=process

[Install]
WantedBy=multi-user.target
```

Убеждаемся что все успешно работает:

```bash
systemctl start spawn-fcgi
systemctl status spawn-fcgi
```
Ссылка на репозиторий с Vagrnatfile и скриптами:
https://github.com/Dekkert/dz8_systemd/tree/master/spawn

 3.Дополнить unit-файл httpd (он же apache) возможностью запустить несколько инстансов сервера с разными конфигурационными файлами.
 
Ссылка на vargrantfile и скрипты:
https://github.com/Dekkert/dz8_systemd/tree/master/httpd

После разворачивании vm можно проверить, что всё поднялось корректно командой
```bash
systemctl status httpd@httpd1.service
systemctl status httpd@httpd2.service
```
