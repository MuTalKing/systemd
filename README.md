# systemd

# Написать сервис, который будет раз в 30 секунд мониторить лог на предмет наличиā ключевого слова. Файл и слово должны задаваться в
/etc/sysconfig

Длā начала создаём файл с конфигурацией длā сервиса в директории
/etc/sysconfig - из неё сервис будет братþ необходимýе переменнýе.
[root@nginx ~#] cat /etc/sysconfig/watchlog
# Configuration file for my watchdog service
# Place it to /etc/sysconfig
# File and word in that file that we will be monit
WORD="ALERT"
LOG=/var/log/watchlog.log
Затем создаем /var/log/watchlog.log и пишем туда строки на своё усмотрение,
плĀс клĀчевое слово ‘ALERT’
