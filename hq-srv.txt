nmcli connection show
nmcli connection del 'Проводное соединение 1'
nmcli connection add con-name “LAN” type ethernet ifname ens192 connection.autoconnect yes ipv4.method manual ipv4.address 192.168.100.1/26 ipv4.gateway 192.168.100.62 ipv4.dns 77.88.8.8
nmcli connection up "LAN"
nmcli connection show

apt-get update
apt-get install nano -y

hostnamectl set-hostname HQ-SRV.au-team.irpo

reboot

control sudo public

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

ssh sshuser@192.168.100.1 -p 2024
yes
P@ssw0rd
sudo ip a

(произвести проверку, зайдя под sshuser, пароль)
НАСТРОЙКА DNS
apt-get update
apt-get install bind bind-utils -y 

nano /etc/bind/options.conf (заполнить только эти строки) (перед строками убрать //)
listen on { 192.168.100.1; };
forwarders { 77.88.8.8; };
allow-query { any; };

nano /etc/bind/rfc1912.conf
zone "au-team.irpo" {
        type master;
        file "au-team.irpo";
	allow-update { none; };
};

zone "100.168.192.in-addr.arpa" {
	type master;
	file "rev1";
	allow-update { none; };
};
zone "200.168.192.in-addr.arpa" {
	type master;
	file "rev2";
	allow-update { none; };
};

cp /etc/bind/zone/localhost /etc/bind/zone/au-team.irpo 
cp /etc/bind/zone/127.in-addr.arpa /etc/bind/zone/rev1
cp /etc/bind/zone/127.in-addr.arpa /etc/bind/zone/rev2

nano /etc/bind/zone/au-team.irpo
@	IN	SOA	au-team.irpo root.au-team.irpo. (
			)
	IN	NS	HQ-SRV.au-team.irpo.
HQ-SRV	IN	A	192.168.100.1
HQ-CLI	IN	A	192.168.200.1
HQ-RTR	IN	A	192.168.100.62
BR-RTR	IN	A	192.168.3.30
BR-SRV	IN	A	192.168.3.1
moodle	IN	A	172.16.4.14
wiki	IN	A	172.16.5.14

nano /etc/bind/zone/rev1
@	IN	SOA	100.168.192.in-addr.arpa. root.au-team.irpo. (
			)
	IN	NS	HQ-SRV.au-team.irpo.
1	IN	PTR	HQ-SRV.au-team.irpo.
62	IN	PTR	HQ-RTR.au-team.irpo.

nano /etc/bind/zone/rev2
@	IN	SOA	200.168.192.in-addr.arpa. root.au-team.irpo. (
			)
	IN	NS	HQ-SRV.au-team.irpo.
1	IN	PTR	HQ-CLI.au-team.irpo.

chown root:named /etc/bind/zone/au-team.irpo
chown root:named /etc/bind/zone/rev1
chown root:named /etc/bind/zone/rev2

nano /etc/resolv.conf
domain au-team.irpo
nameserver 192.168.100.1

systemctl enable bind --now
systemctl restart bind

(проверка)
nslookup hq-srv.au-team.irpo
(должно выйти)
server: 192.168.100.1
address 192.168.100.1#53
nslookup hq-cli.au-team.irpo
nslookup br-srv.au-team.irpo

timedatectl set-timezone Europe/Moscow
timedatectl show

Модуль 2
nano /etc/bind/options.conf (заполнить только эти строки)
forwarders { 77.88.8.8; 192.168.3.1; };
dnssec-validation no; (добавить в options, последней строчкой перед };)

systemctl restart bind

Добавить три диска по 1 гб в веб интерфейсе

lsblk (проверяем, что новые диски видны)
mdadm --zero-superblock --force /dev/sd{b,c,d} (результат не важен, но сделать надо)
wipefs --all --force /dev/sd{b,c,d}
mdadm --create /dev/md0 -l 5 -n 3 /dev/sd{b,c,d}
lsblk (под дисками должно появиться md0)
mkfs -t ext4 /dev/md0 (если спросит - Y)
mkdir /etc/mdadm
echo "DEVICE partitions" > /etc/mdadm/mdadm.conf
mdadm --detail --scan | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf
mkdir /mnt/raid5

nano /etc/fstab
/dev/md0	/mnt/raid5	ext4	defaults	0	0

mount -a
df -h (проверка, должно вывести среди всего /dev/md0)
(лучше сделать снапшот)
make-initrd
make-initrd
reboot

apt-get install -y nfs-{server,utils}
mkdir /mnt/raid5/nfs
chmod 766 /mnt/raid5/nfs

nano /etc/exports
/mnt/raid5/nfs 192.168.200.0/28(rw,no_root_squash)

exportfs -arv
systemctl enable --now nfs-server
systemctl restart nfs-server

apt-get install chrony -y

nano /etc/chrony.conf
#pool pool.ntp.org iburst (закомментировать)
server 172.16.4.14 iburst prefer

systemctl enable --now chronyd
systemctl restart chronyd

chronyc sources (172.16.4.14)

apt-get install -y lamp-server
apt-get install -y  php7-mbstring php7-gd php7-xmlreader php7-zip php7-intl php7-openssl php7-curl php7-fileinfo php7-sodium php7-soap php7-exif

nano /etc/my.cnf.d/server.cnf
[mysqld]
user	= mysql
default_storage_engine = innodb
innodb_file_per_table = 1
innodb_file_format = Barracuda

systemctl enable --now mysqld
systemctl restart mysqld

mysql -u root
CREATE DATABASE moodledb;
CREATE USER 'moodle'@'localhost' IDENTIFIED BY 'P@ssw0rd';
GRANT ALL ON moodledb.* TO 'moodle'@'localhost';
FLUSH PRIVILEGES;
quit

cd /home
curl -LO https://download.moodle.org/download.php/direct/stable401/moodle-latest-401.zip
unzip moodle-latest-401.zip -d /var/www/html/
cd

mkdir /var/www/html/moodledata
chown -R apache2:apache2 /var/www/html/moodle/
chmod -R 755 /var/www/html/moodle/
chown -R apache2:apache2 /var/www/html/moodledata/

nano /etc/httpd2/conf/sites-available/moodle.conf
<VirtualHost *:80>
    ServerAdmin root@localhost
    ServerName hq-srv.au-team.irpo
    ServerAlias moodle.au-team.irpo
    DocumentRoot /var/www/html/moodle

    <Directory /var/www/html/moodle/>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    ErrorLog /var/log/httpd2/nerror_log
    CustomLog /var/log/httpd2/naccess_log combined
</VirtualHost>

nano /etc/php/7.4/apache2-mod_php/php.ini
строчка над ориентиром Error handling and logging
 max_input_vars = 6000 (расскоментрировать и отредактировать)

a2ensite moodle

systemctl restart httpd2
systemctl enable --now httpd2




