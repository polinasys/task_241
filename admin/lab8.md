Шарим
1. Установите пакет samba
su -
apt-get update
apt-get install samba

2. ЧТо такое общая папка, зачем оно может быть нужно?
Общая папка - это директория, доступная по сети для других пользователей/компьютеров
Используется для:
- Обмена файлами между компьютерами
- Совместной работы над проектами
- Резервного копирования
- Доступа к файлам с разных устройств

3. Создайте общую папку без пароля с правами только на чтение файлов
su -
mkdir -p /samba/public
chmod 755 /samba/public
echo "test file" > /samba/public/readme.txt
nano /etc/samba/smb.conf

Добавляем в конец файла:
[public]
    path = /samba/public
    browseable = yes
    read only = yes
    guest ok = yes

systemctl restart smb

4. Создайте общую папку с паролем с правами на чтение и запись
mkdir -p /samba/private
chmod 777 /samba/private
useradd -M sambauser
smbpasswd -a sambauser
nano /etc/samba/smb.conf

Добавляем в конец файла:
[private]
    path = /samba/private
    browseable = yes
    read only = no
    guest ok = no
    valid users = sambauser

systemctl restart smb

5. Создайте общую папку с доступом для какой-то группы с полными правами
groupadd smbgroup
mkdir -p /samba/group
chgrp smbgroup /samba/group
chmod 770 /samba/group
useradd -G smbgroup user1
smbpasswd -a user1
nano /etc/samba/smb.conf

Добавляем в конец файла:
[group]
    path = /samba/group
    browseable = yes
    read only = no
    guest ok = no
    valid users = @smbgroup
    write list = @smbgroup

systemctl restart smb

6. Создайте общую папку в которой у одной группы будет полный доступ, а у другой только доступ на чтение. Третья группа не должна иметь к ней доступа
groupadd fullaccess
groupadd readonly
groupadd noaccess
mkdir -p /samba/mixed
chgrp fullaccess /samba/mixed
chmod 775 /samba/mixed
useradd -G fullaccess user2
useradd -G readonly user3
useradd -G noaccess user4
smbpasswd -a user2
smbpasswd -a user3
smbpasswd -a user4
nano /etc/samba/smb.conf

[mixed]
    path = /samba/mixed
    browseable = yes
    read only = no
    guest ok = no
    valid users = @fullaccess, @readonly
    write list = @fullaccess
    read list = @readonly
    invalid users = @noaccess

systemctl restart smb

# Дополняю секцию global !
    map to guest = bad user
    guest account = nobody 

Разрешаю анонимный гостевой доступ к общей папке public без ввода пароля 

Проверяем, что все работает:
smbclient //localhost/public -N

Вывод: Try "help" to get a list of possible commands.
smb: \> help
?              allinfo        altname        archive        backup         
blocksize      cancel         case_sensitive cd             chmod          
chown          close          del            deltree        dir            
du             echo           exit           get            getfacl        
geteas         hardlink       help           history        iosize         
lcd            link           lock           lowercase      ls             
l              mask           md             mget           mkdir          
mkfifo         more           mput           newer          notify         
open           posix          posix_encrypt  posix_open     posix_mkdir    
posix_rmdir    posix_unlink   posix_whoami   print          prompt         
put            pwd            q              queue          quit           
readlink       rd             recurse        reget          rename         
reput          rm             rmdir          showacls       setea          
setmode        scopy          stat           symlink        tar            
tarmode        timeout        translate      unlock         volume         
vuid           wdel           logon          listconnect    showconnect    
tcon           tdis           tid            utimes         logoff         
..             !     
smb: \> pwd
Current directory is \\localhost\public\