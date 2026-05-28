hostnamectl set-hostname BR-RTR.au-team.irpo

reboot

nano /etc/network/interfaces
auto ens192
iface ens192 inet static
address 172.16.5.1/28
gateway 172.16.5.14
auto ens224
iface ens224 inet static
address 192.168.3.30/27

systemctl restart networking

nano /etc/sysctl.conf
net.ipv4.ip_forward=1 (раскомментировать)

sysctl -p 

touch gre.up
chmod +x gre.up

nano ./gre.up
#!/bin/bash
ip tunnel add gre1 mode gre remote 172.16.4.1 local 172.16.5.1 ttl 255
ip link set gre1 up
ip addr add 10.255.255.2/30 dev gre1

nano /etc/crontab
@reboot root /root/gre.up

nano /etc/resolv.conf
domain localdomain
search localdomain
nameserver 77.88.8.8

nano /etc/apt/sources.list
deb http://mirror.yandex.ru/debian bookworm main contrib
deb-src http://mirror.yandex.ru/debian bookworm main contrib

apt update 
apt install frr –y

nano /etc/frr/daemons
ospfd=yes

systemctl restart frr
vtysh
conf te
router ospf
network 10.255.255.0/30 area 0.0.0.0
network 192.168.3.0/27 area 0.0.0.0
exit
int gre1
ip ospf network point-to-point
exit
exit
wr
exit
systemctl restart frr

nano ./gre.up
sysctl -p
systemctl restart frr

reboot

ip ro

apt update
apt install iptables-persistent -y

iptables -t nat -A POSTROUTING -o ens192 -j MASQUERADE
iptables-save >> /etc/iptables/rules.v4
systemctl restart iptables

timedatectl set-timezone Europe/Moscow
timedatectl show

useradd net_admin -u 1010 -d /home/net_admin -m -G root -s /bin/bash
passwd net_admin (P@ssw0rd)

apt install sudo -y

nano /etc/sudoers
root ALL=(ALL:ALL) ALL (ориентир)
net_admin ALL=(ALL:ALL) NOPASSWD: ALL

Модуль 2

apt-get install -y chrony

nano /etc/chrony/chrony.conf
#pool 2.debian.pool.ntp.org iburst (закомментировать)
server 172.16.5.14 iburst prefer

systemctl enable --now chrony
systemctl restart chrony

chronyc sources (172.16.5.14)

apt install openssh-server -y
systemctl daemon-reload

nano /etc/ssh/sshd_config
Port 22
AllowUsers	net_admin
PasswordAuthentication yes

systemctl restart ssh
systemctl enable ssh
systemctl status ssh

ssh net_admin@192.168.3.30 -p 22
yes
P@ssw0rd
sudo ip a

iptables -t nat -A PREROUTING -i ens192 -p tcp --dport 80 -j DNAT --to-destination 192.168.3.1:8080
iptables -t nat -A PREROUTING -i ens192 -p tcp --dport 2024 -j DNAT --to-destination 192.168.3.1:2024
iptables -t nat -A POSTROUTING -o ens192 -j MASQUERADE
iptables-save >> /etc/iptables/rules.v4
systemctl restart iptables
