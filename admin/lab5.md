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
su -

mkdir -p /usr/local/bin
nano /usr/local/bin/system_info_collector.sh
# Зписываем в открывшийся файла:
#!/bin/bash
set -euo pipefail

# Всегда переходим в домашнюю директорию пользователя
cd ~

# Проверяем и создаем папку
if [ ! -d "system_info_data" ]; then
    mkdir -p system_info_data
    echo "Создана папка system_info_data"
else
    echo "Папка system_info_data уже существует"
fi

# Создаем 4 файла с информацией
for i in 1 2 3 4; do
    file_path="system_info_data/file$i.txt"
    if [ -f "$file_path" ]; then
        echo "Файл $file_path уже существует, перезаписываем"
    fi
    
    cat > "$file_path" << EOF
Дата: $(date)
Ядро: $(uname -r)
Компьютер: $(hostname)
Пользователь: $(whoami)

Домашняя папка:
$(ls -la ~)
EOF
    echo "Создан файл $file_path"
done

echo "Скрипт выполнен успешно"

# Делаем скрипт исполняемым
chmod +x /usr/local/bin/system_info_collector.sh

2. Создайте юнит, который будет вызывать этот скрипт при запуске. Проверьте
su -
nano /etc/systemd/system/system-info.service

# Записываем
[Unit]
Description=System Info Collector

[Service]
Type=oneshot
ExecStart=/usr/local/bin/system_info_collector.sh
User=root

[Install]
WantedBy=multi-user.target

# Тестируем
systemctl daemon-reload
systemctl start system-info.service
systemctl status system-info.service

3. Создайте таймер который будет вызывать выполнение одноимённого systemd юнита каждые 5 минут.
su -
nano /etc/systemd/system/system-info.timer

# Записываем
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
su -
useradd -m systemuser

6. Дополните юнит информацией о пользователе, от которого должен выполняться скрипт.
su -
nano /etc/systemd/system/system-info.service

# Обновляем
[Unit]
Description=System Info Collector

[Service]
Type=oneshot
ExecStart=/usr/local/bin/system_info_collector.sh
User=systemuser

[Install]
WantedBy=multi-user.target

# Обновляем
systemctl daemon-reload
chown systemuser:systemuser /usr/local/bin/system_info_collector.sh

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