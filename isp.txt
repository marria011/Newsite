nano /etc/network/interfaces
auto ens224 
iface ens224 inet static
address 172.16.5.14
netmask 255.255.255.240
auto ens256
iface ens256 inet static
address 172.16.4.14
netmask 255.255.255.240

systemctl restart networking

nano /etc/sysctl.conf
net.ipv4.ip_forward=1 (раскомментировать)

sysctl -p 

hostnamectl set-hostname ISP

reboot

nano /etc/apt/sources.list
deb http://mirror.yandex.ru/debian bookworm main contrib
deb-src http://mirror.yandex.ru/debian bookworm main contrib

apt update
apt install iptables-persistent -y

iptables -t nat -A POSTROUTING -o ens192 -j MASQUERADE
iptables-save >> /etc/iptables/rules.v4
systemctl restart iptables

timedatectl set-timezone Europe/Moscow
timedatectl show

Модуль 2

apt-get install -y chrony

nano /etc/chrony/chrony.conf
#pool 2.debian.pool.ntp.org iburst (закомментировать)
server 127.0.0.1 iburst prefer
local stratum 5
allow 0/0

systemctl enable --now chrony
systemctl restart chrony

chronyc tracking (stratum 5)
chronyc sources (localhost должен быть)

apt install nginx -y 

nano /etc/nginx/sites-enabled/default
nano /etc/nginx/sites-available/proxy.conf
server {
    listen 80;
    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;

    server_name moodle.au-team.irpo;

    location / {
        proxy_pass http://172.16.4.1;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

server {
    listen 80;
    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;

    server_name wiki.au-team.irpo;

    location / {
        proxy_pass http://172.16.5.1;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
