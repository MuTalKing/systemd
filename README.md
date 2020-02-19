# systemd

# Написать сервис, который будет раз в 30 секунд мониторить лог на предмет наличиā ключевого слова. Файл и слово должны задаваться в /etc/sysconfig

Длā начала создаём файл с конфигурацией длā сервиса в директории /etc/sysconfig - из неё сервис будет брать необходимые переменные.
```
[root@nginx ~#] cat /etc/sysconfig/watchlog
# Configuration file for my watchdog service
# Place it to /etc/sysconfig
# File and word in that file that we will be monit
WORD="ALERT"
LOG=/var/log/watchlog.log
```
Затем создаем /var/log/watchlog.log и пишем туда строки на своё усмотрение, плюс ключевое слово ‘ALERT’

Создадим скрипт:
```
[root@nginx ~#] cat /opt/watchlog.sh
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
Команда logger отправлāет лог в системный журнал

Создадим юнит для сервиса:
```
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
```
[Unit]
Description=Run every 30 seconds

[Timer]
OnBootSec=1m
OnUnitActiveSec=30s
Unit=watchlog.service

[Install]
WantedBy=timers.target
```

Затем достаточно только стартануть timer:

**[root@nginx ~#] systemctl start watchlog.timer**

И убедиться в результате:

![Image alt](https://github.com/MuTalKing/systemd/blob/master/watchlog.jpg)

# Из epel установить spawn-fcgi и переписать init-скрипт на unit-файл. Имя сервиса должно также называться.

Устанавливаем spawn-fcgi и необходимые для него пакеты:

[root@nginx ~#] yum install epel-release -y && yum install spawn-fcgi php php-climod_fcgid httpd -y

etc/rc.d/init.d/spawn-fcg - cам Init скрипт, который будем переписывать

Но перед этим необходимо раскомментировать строки с переменными в /etc/sysconfig/spawn-fcgi

Он должен получиться следующего вида:

```
[root@nginx ~#] cat /etc/sysconfig/spawn-fcgi
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

[root@nginx ~#] cat /etc/systemd/system/spawn-fcgi.service
```
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

![Image alt](https://github.com/MuTalKing/systemd/blob/master/spawn-fcgi.jpg)

# Дополнить юнит-файл apache httpd возможностью запустить несколько инстансов сервера с разными конфигами

![Image alt](https://github.com/MuTalKing/systemd/blob/master/httpd%7B1%2C2%7D.jpg)
