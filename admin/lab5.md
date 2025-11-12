Task 1
Юниты
1. Что такое systemd юнит?
Systemd юнит - это конфигурационный файл, который описывает как systemd должен управлять службой, сокетом, устройством, точкой монтирования и другими системными объектами.

Основные типы юнитов:
service - системные службы
socket - сокеты для активации по требованию
timer - планировщик задач (замена cron)
mount - точки монтирования
target - группы юнитов (аналог runlevels)

2. Проверье статус любого systemd юнита, какую информацию выводит эта команда?
su -
systemctl status sshd 

Эта команда показывает статус службы SSH демона (sshd):
Состояние "active (running)"
PID процесса
Время работы
Последние записи в логах

3. ПОпробуйте оставновить сервис.
systemctl stop sshd

Проверяем:
systemctl status sshd

4. Перезапустите его.
systemctl restart sshd

5. УДалите из автозагрузки
systemctl disable sshd

6. Верните обратно
systemctl enable sshd

7. Что такое таймеры?
Таймеры - это systemd юниты для планирования выполнения задач (аналог cron). Они могут запускать service юниты по расписанию.

Task 2
Пишем юниты
1. Создайте скрипт который создаёт папку заполняет её файлами ( имена 1-4 ) и записывает в них информацию о текущей дате, версии ядра, имени компьютера и списе всех файлов в домашнем каталоге пользователя от которого выполняется скрипт( не забудьте сдлеать проверку на существование файлов и папок)
#!/bin/bash
set -euo pipefail

cd ~

mkdir -p system_info_data

for i in 1 2 3 4; do
    cat > "system_info_data/file$i.txt" << EOF
Дата: $(date)
Ядро: $(uname -r)
Компьютер: $(hostname)
Пользователь: $(whoami)

Домашняя папка:
$(ls -la ~)
EOF
done

2. Создайте юнит, который будет вызывать этот скрипт при запуске. Проверьте
[Unit]
Description=System Info Collector

[Service]
Type=oneshot
ExecStart=/usr/local/bin/system_info_collector.sh
User=root

[Install]
WantedBy=multi-user.target

3. Создайте таймер который будет вызывать выполнение одноимённого systemd юнита каждые 5 минут.
[Unit]
Description=Run every 5 minutes

[Timer]
OnBootSec=1min
OnUnitActiveSec=5min

[Install]
WantedBy=timers.target

4. От какого пользователя вызыаются юниты по умолчанию?
По умолчанию от root'а

5. Создайте пользователя, от имени которого будет выполняться ваш скрипт.
sudo useradd -m systemuser

6. Дополните юнит информацией о пользователе, от которого должен выполняться скрипт.
[Unit]
Description=System Info Collector

[Service]
Type=oneshot
ExecStart=/usr/local/bin/system_info_collector.sh
User=systemuser

[Install]
WantedBy=multi-user.target

7. Дополните ваш скрипт так, чтобы он независимо от местоположения всегда выполнялся в домашней папке того, кто его вызывает.
cd ~ # Выполняется из домашней директории.
...

Task 3
Журнальчики
1. Посмотретите журналы ssh
su -
journalctl -u sshd

2. Выведите журналы в реальном времени
journalctl -f

3. Выведите лог в реальном времени для службы sshd
journalctl -u sshd -f

4. Можно ли без комады journalctl прочитать логи systemd?
Да, можно несколькими способами:
su -
# 1. Через системные файлы
cat /var/log/syslog | grep ssh
# 2. Через службу rsyslog
cat /var/log/auth.log | grep ssh
# 3. Через демона службы (если ведет собственный лог)
cat /var/log/ssh/*
# 4. Просмотр бинарных журналов напрямую
strings /var/log/journal/*/system.journal | grep ssh

5. Сколько будет 2-2?
echo $((2 - 2))
# Результат: 0