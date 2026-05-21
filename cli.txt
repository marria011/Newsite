(после настройки dhcp на hq-rtr)

su-
apt-get update
apt-get install nano -y

hostnamectl set-hostname HQ-CLI.au-team.irpo

reboot

Модуль 2
Поставить Только автоматические адреса (DHCP)
Поменять DNS на 192.168.3.1
Зайти в центр управления системой
Выбрать аутентификация, поставить домен Active Directory
Домен: au-team.irpo
Рабочая группа: AU-TEAM
Имя компьютера: HQ-CLI

В терминале:
cd /home

nano users.csv
username,password
user1.hq,P@ssw0rd
user2.hq,P@ssw0rd
user3.hq,P@ssw0rd
user4.hq,P@ssw0rd
user5.hq,P@ssw0rd

scp -P 2024 users.csv sshuser@192.168.3.1:/home/sshuser
yes
P@ssw0rd

В новом окне терминала
control sudo public
su -

nano /etc/sudoers
##
## User privillage specification
##
 root ALL=(ALL) ALL (расскоментировать)
# WHEEL_USERS ALL=(ALL) ALL (ориентир)
 %hq ALL=(ALL) NOPASSWD: /usr/bin/cat, /usr/bin/grep, /usr/bin/id

Дальше зайти под доменным пользователем и проверить
sudo ip a (не должно быть вывода)
sudo id (вывод будет)

Под админом после настройки nfs
su -
apt-get install -y nfs-{utils,clients}
mkdir /mnt/nfs
chmod 777 /mnt/nfs

nano /etc/fstab
192.168.100.1:/mnt/raid5/nfs  /mnt/nfs  nfs  defaults  0  0

mount -a
df -h (должен появиться с ip)

apt-get install chrony -y

nano /etc/chrony.conf
#pool AU-TEAM.IRPO iburst (закомментировать)
server 172.16.4.14 iburst prefer

systemctl enable --now chronyd
systemctl restart chronyd

chronyc sources (172.16.4.14)

useradd -u 1010 sshuser
passwd sshuser (установить пароль P@ssw0rd)

nano /etc/sudoers
 root ALL=(ALL:ALL) ALL (ОРИЕНТИР)
 sshuser ALL=NOPASSWD: ALL

gpasswd -a sshuser wheel

apt-get install openssh-server -y
systemctl daemon-reload

nano /etc/openssh/sshd_config
Port 2024
MaxAuthTries 2
PasswordAuthentication yes
AllowUsers	sshuser
Banner /etc/openssh/banner

nano /etc/openssh/banner
Authorized access only!!!

systemctl restart sshd
systemctl enable sshd --now
systemctl status sshd

ssh sshuser@(вставь IP, вместо скобок) -p 2024
yes
P@ssw0rd
sudo ip a

Заходим в браузер
192.168.3.1:8080
язык ru
далее
mariadb
хост базы данных: wiki-database-1
имя базы данных: mediawiki
имя пользователя базы данных: wikiuser
пароль базы данных: P@ssw0rd
далее
далее
название вики: wiki
пространство имен проекта: то же, что имя вики
Ваше имя участника: admin
пароль: P@ssw0rd91
адрес электронной почты: admin@au-team.irpo
Хватит уже, просто установити вики
далее
далее
далее
Скачался файл
Переходим в консоль
su -
cd /home/AU-TEAM.IRPO/administrator/Загрузки
ls (проверям есть ли файл, если нет смотрим путь через загрузки)
scp -P 2024 LocalSettings.php sshuser@192.168.3.1:/home/sshuser
P@ssw0rd

После внесения изменений на сервере, возвращаемся на клиента и в браузере http://br-srv.au-team.irpo:8080/

после добавления dns записси на BR-SRV
в браузере
http://192.168.100.1
язык en
next
next
mariadb next
database host: localhost
database name: moodledb
Пользователей базы данных: moodle
Пароль: P@ssw0rd
tables prefix: mdl_ 
contnue
continue
continue
continue
Username: admin
New password: P@ssw0rd
email: admin@au-team.irpo

country: Russian Federation
timezone: Europe/Moscow
update profile
Full site name: (твое рабочее место)
Short name for site: (твое рабочее место)
Default timezone: Europe/Moscow
Support email: admin@au-team.irpo
No-reply address: noreply@au-team.irpo
save changess

установка яндекс браузера через консоль
su -
apt-get install -y yandex-browser-stable



