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
WORD=$1
LOG=$2
DATE=`date`
if grep $WORD $LOG &> /dev/null
then
logger "$DATE: I found word, Master!"
else
exit 0
fi
```
Команда logger отправлāет лог в системный журнал

Создадим юнит для сервиса:
```
[Unit]
Description=My watchlog service
[Service]
Type=oneshot
EnvironmentFile=/etc/sysconfig/watchdog
ExecStart=/opt/watchlog.sh $WORD $LOG
```

Создадим юнит для таймера:
```
[Unit]
Description=Run watchlog script every 30 second
[Timer]
# Run every 30 second
OnUnitActiveSec=30
Unit=watchlog.service
[Install]
WantedBy=multi-user.target
```

Затем достаточно толþко стартануть timer:
**[root@nginx ~#] systemctl start watchlog.timer**
И убедиться в результате:

![Image alt](https://github.com/MuTalKing/systemd/blob/master/watchlog.jpg)


