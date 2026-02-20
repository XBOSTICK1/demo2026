# demo2025
## Модуль 1
### Распределение IP:
| Имя устройства |  IP адресс     | Шлюз         |
| -------------- | ----------     |  ----------- |
|HQ-RTR          |172.16.1.2/28   |172.16.2.1    |
|BR-RTR		       |172.16.2.2/28   |172.16.2.1    |
|HQ-SRV          |192.168.100.2/27|192.168.100.1 |
|HQ-CLI          |DHCP/27         |192.168.200.1 |
|BR-SRV          |192.168.1.2/28  |192.168.1.1   |
## Задание 1. Выдача имени устройству:
```
В ISP пишем - hostnamectl set-hostname ISP; exec bash
В HQ-RTR пишем - hostnamectl set-hostname HQ-SRV.au-team.irpo; exec bash
В BR-RTR пишем - hostnamectl set-hostname HQ-SRV.au-team.irpo; exec bash
В HQ-SRV пишем - hostnamectl set-hostname HQ-SRV.au-team.irpo; exec bash
В BR-SRV пишем - hostnamectl set-hostname BR-SRV.au-team.irpo; exec bash
В HQ-CLI пишем - hostnamectl set-hostname HQ-CLI.au-team.irpo; exec bash 
```
## Задание 2. Назначение IP: (ПЖ прежде сделать посмотрите интерфейсы (адаптеры) в esxi или proxmox виртуальных машинах в настройках)
### ISP
Просмотр и создание директорий для интерфейсов
```
ip -c a
mkdir /etc/net/ifaces/ens34
mkdir /etc/net/ifaces/ens35
```
Создание настроек для интерфейсов
```
vim /etc/net/ifaces/ens34/options
TYPE=eth
DISABLED=no
NM_CONTROLLED=no
BOOTPROTO=static
CONFIG_IPv4=yes
```
```
vim /etc/net/ifaces/ens35/options
TYPE=eth
DISABLED=no
NM_CONTROLLED=no
BOOTPROTO=static
CONFIG_IPv4=yes
```
Выдача IP
```
echo 172.16.1.1/28 > /etc/net/ifaces/ens34/ipv4address
echo 172.16.2.1/28 > /etc/net/ifaces/ens35/ipv4address
```
включение forwarding, в строке net.ipv4.ip_forward поменять 0 на 1
```
vim /etc/net/sysctl.conf
```
Перезагрузка службы network
```
systemctl restart network
```
### HQ-RTR
```
ip -c a

mkdir /etc/net/ifaces/ens33
mkdir /etc/net/ifaces/ens34
mkdir /etc/net/ifaces/ens35
```
```
vim /etc/net/ifaces/ens33/options
TYPE=eth
DISABLED=no
NM_CONTROLLED=no
BOOTPROTO=static
CONFIG_IPv4=yes
```
и повторите для ens34, ens35
Выдаем IP
```
echo "172.16.1.2/28" > /etc/net/ifaces/ens33/ipv4address
echo "default via 172.16.1.1" > /etc/net/ifaces/ens33/ipv4route
echo "192.168.100.1/27 > /etc/net/ifaces/ens34/ipv4address
echo "192.168.200.1/28 > /etc/net/ifaces/ens35/ipv4address
```
включение forwarding, в строке net.ipv4.ip_forward поменять 0 на 1
```
vim /etc/net/sysctl.conf
```
Выдаем ДНС (смотрите какой днс у ISP)
```
vim /etc/resolv.conf
nameserver (ваш днс сервер) 
```
Перезагрузка службы network
```
systemctl restart network
```
### BR-RTR
```
ip -c a

mkdir /etc/net/ifaces/ens33
mkdir /etc/net/ifaces/ens34
```
```
vim /etc/net/ifaces/ens33/options
TYPE=eth
DISABLED=no
NM_CONTROLLED=no
BOOTPROTO=static
CONFIG_IPv4=yes
```
и повторите для ens34
Выдаем IP
```
echo "172.16.2.2/28" > /etc/net/ifaces/ens33/ipv4address
echo "default via 172.16.2.1" > /etc/net/ifaces/ens33/ipv4route
echo "192.168.1.1/28 > /etc/net/ifaces/ens34/ipv4address
```
включение forwarding, в строке net.ipv4.ip_forward поменять 0 на 1
```
vim /etc/net/sysctl.conf
```
Выдаем ДНС (смотрите какой днс у ISP)
```
vim /etc/resolv.conf
nameserver (ваш днс сервер) 
```
Перезагрузка службы network
```
systemctl restart network
```
## HQ-SRV
```
vim /etc/net/ifaces/ens33/options
TYPE=eth
DISABLED=no
NM_CONTROLLED=no
BOOTPROTO=static
CONFIG_IPv4=yes
```
Выдаем IP
```
echo "192.168.100.1/28" > /etc/net/ifaces/ens33/ipv4address
echo "default via 192.168.100.1" > /etc/net/ifaces/ens33/ipv4route
```
включение forwarding, в строке net.ipv4.ip_forward поменять 0 на 1
```
vim /etc/net/sysctl.conf
```
Выдаем ДНС (смотрите какой днс у ISP)
```
vim /etc/resolv.conf
nameserver (ваш днс сервер) 
```
Перезагрузка службы network
```
systemctl restart network
```
## BR-SRV
```
vim /etc/net/ifaces/ens33/options
TYPE=eth
DISABLED=no
NM_CONTROLLED=no
BOOTPROTO=static
CONFIG_IPv4=yes
```
Выдаем IP
```
echo "192.168.1.2/28" > /etc/net/ifaces/ens33/ipv4address
echo "default via 192.168.1.1" > /etc/net/ifaces/ens33/ipv4route
```
включение forwarding, в строке net.ipv4.ip_forward поменять 0 на 1
```
vim /etc/net/sysctl.conf
```
Выдаем ДНС (смотрите какой днс у ISP)
```
vim /etc/resolv.conf
nameserver (ваш днс сервер) 
```
Перезагрузка службы network
```
systemctl restart network
```
##  Настройка NAT
### ISP
Установка firewalld:
```
apt-get update && apt-get -y install firewalld && systemctl enable --now firewalld
```
Правила к исходящим пакетам (в сторону провайдера):
```
firewall-cmd --permanent --zone=public --add-interface=ens33
```
Правила к входящим пакетам (к локальной сети):
```
firewall-cmd --permanent --zone=trusted --add-interface=ens34
firewall-cmd --permanent --zone=trusted --add-interface=ens35
```
Включение NAT:
```
firewall-cmd --permanent --zone=public --add-masquerade
```
Сохранение правил:
```
firewall-cmd --complete-reload
```
включим forward
```
vim /etc/net/sysctl.conf
net.ipv4.ip_forward = 1
```
### HQ-RTR

Установка firewalld:
```
apt-get update && apt-get -y install firewalld && systemctl enable --now firewalld
```
Правила к исходящим пакетам (в сторону провайдера):
```
firewall-cmd --permanent --zone=public --add-interface=ens33
```
Правила к входящим пакетам (к локальной сети):
```
firewall-cmd --permanent --zone=trusted --add-interface=ens34
firewall-cmd --permanent --zone=trusted --add-interface=ens35
```
Включение NAT:
```
firewall-cmd --permanent --zone=public --add-masquerade
```
Сохранение правил:
```
firewall-cmd --complete-reload
```
включим forward
```
vim /etc/net/sysctl.conf
net.ipv4.ip_forward = 1
```
### BR-RTR

Установка firewalld:
```
apt-get update && apt-get -y install firewalld && systemctl enable --now firewalld
```
Правила к исходящим пакетам (в сторону провайдера):
```
firewall-cmd --permanent --zone=public --add-interface=ens33
```
Правила к входящим пакетам (к локальной сети):
```
firewall-cmd --permanent --zone=trusted --add-interface=ens34
```
Включение NAT:
```
firewall-cmd --permanent --zone=public --add-masquerade
```
Сохранение правил:
```
firewall-cmd --complete-reload
```
включим forward
```
vim /etc/net/sysctl.conf
net.ipv4.ip_forward = 1
```
## Задание 3. Создание локальных учетных записей HQ-SRV, BR-SRV

#### HQ-SRV
Создание учетных записей
```
useradd -m -u 2026 sshuser
passwd sshuser
Пароль: P@ssw0rd
```
Далее идет команды внутри sshuser
Проверка
```
su - sshuser
whoami
```

ПОВТОРЯЕМ ТОЖЕ САМОЕ В BR-SRV!!
## Задание 4. Настройка VLAN на HQ-RTR (VLAN 100, 200, 999)
Сперва выдаем IP-адреса для HQ-SRV (На HQ-CLI будет DHCP)
Создаем каталог для под интерфейса:
```
mkdir /etc/net/ifaces/ens34.100
mkdir /etc/net/ifaces/ens35.200
mkdir /etc/net/ifaces/ens33.999
```
Далее настраиваем OPTIONS
Для VLAN 100
```
TYPE=vlan
HOST=ens34
VID=100
DISABLED=no
BOOTPROTO=static
ONBOOT=yes
CONFIG_IPV4=yes
```
Для VLAN 200
```
TYPE=vlan
HOST=ens35
VID=200
DISABLED=no
BOOTPROTO=static
ONBOOT=yes
CONFIG_IPV4=yes
```
Для VLAN 999
```
TYPE=vlan
HOST=ens33
VID=999
DISABLED=no
BOOTPROTO=static
ONBOOT=yes
CONFIG_IPV4=yes
```
Выдаем IP-адреса
```
echo "192.168.10.1/27" > /etc/net/ifaces/ens34.100/ipv4address
echo "192.168.20.1/28" > /etc/net/ifaces/ens34.999/ipv4address
echo "192.168.99.1/29" > /etc/net/ifaces/ens34.999/ipv4address
```
## Задание 5. Настройка безопасный удаленный доступ на серверах HQ-SRV и BR-SRV
### Делаем на обеих машинах (HQ-SRV, BR-SRV)
Скачиваем SSH SERVER
``` 
apt-get update && apt-get install openssh-server -y
```
Далее заходим в конфиг
```
vim /etc/openssh/sshd_config
```
И пишем
```
Port 2026
MaxAuthTries 2
Banner /etc/openssh/sshd_banner
AllowUsers sshuser
```
Далее настроим баннер
```
vim /etc/openssh/sshd_banner
«Authorized access only»
```
Далее запускаем
```
systemctl enable sshd --now
systemctl restart sshd

systemctl status sshd
```
## Задание 6. Настройка IP-туннель на HQ-RTR и BR-RTR
### HQ-RTR и BR-RTR
Сперва заходим в настройка модулей
```
vim /etc/modules
```
и пишем снизу
```
tun
gre
```
### HQ-RTR
Далее создаем каталог 
```
mkdir /etc/net/ifaces/gre1
vim /etc/net/ifaces/gre1/options

TYPE=iptun
TUNTYPE=gre
TUNLOCAL=172.16.1.2
TUNREMOTE=172.16.2.2
TUNOPTIONS='ttl 64'
HOST=ens33
```
далее выдаем IP
```
echo "10.10.10.1/30" > /etc/net/ifaces/gre1/ipv4address
```
### BR-RTR
Далее создаем каталог 
```
mkdir /etc/net/ifaces/gre1
vim /etc/net/ifaces/gre1/options

TYPE=iptun
TUNTYPE=gre
TUNLOCAL=172.16.2.2
TUNREMOTE=172.16.1.2
TUNOPTIONS='ttl 64'
HOST=ens33
```
далее выдаем IP
```
echo "10.10.10.2/30" > /etc/net/ifaces/gre1/ipv4address
```
##  Задание 7: Обеспечьте динамическую маршрутизацию: ресурсы одного офиса должны быть доступны из другого офиса. Для обеспечения динамической маршрутизации используйте link state протокол на ваше усмотрение.
Я хз это как сделать, вот <a href = "https://github.com/meowehh/DemoExam_2026/blob/main/Module_1.md#-%D0%B7%D0%B0%D0%B4%D0%B0%D0%BD%D0%B8%D0%B5-7-%D0%BE%D0%B1%D0%B5%D1%81%D0%BF%D0%B5%D1%87%D1%8C%D1%82%D0%B5-%D0%B4%D0%B8%D0%BD%D0%B0%D0%BC%D0%B8%D1%87%D0%B5%D1%81%D0%BA%D1%83%D1%8E-%D0%BC%D0%B0%D1%80%D1%88%D1%80%D1%83%D1%82%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D1%8E-%D1%80%D0%B5%D1%81%D1%83%D1%80%D1%81%D1%8B-%D0%BE%D0%B4%D0%BD%D0%BE%D0%B3%D0%BE-%D0%BE%D1%84%D0%B8%D1%81%D0%B0-%D0%B4%D0%BE%D0%BB%D0%B6%D0%BD%D1%8B-%D0%B1%D1%8B%D1%82%D1%8C-%D0%B4%D0%BE%D1%81%D1%82%D1%83%D0%BF%D0%BD%D1%8B-%D0%B8%D0%B7-%D0%B4%D1%80%D1%83%D0%B3%D0%BE%D0%B3%D0%BE-%D0%BE%D1%84%D0%B8%D1%81%D0%B0-%D0%B4%D0%BB%D1%8F-%D0%BE%D0%B1%D0%B5%D1%81%D0%BF%D0%B5%D1%87%D0%B5%D0%BD%D0%B8%D1%8F-%D0%B4%D0%B8%D0%BD%D0%B0%D0%BC%D0%B8%D1%87%D0%B5%D1%81%D0%BA%D0%BE%D0%B9-%D0%BC%D0%B0%D1%80%D1%88%D1%80%D1%83%D1%82%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D0%B8-%D0%B8%D1%81%D0%BF%D0%BE%D0%BB%D1%8C%D0%B7%D1%83%D0%B9%D1%82%D0%B5-link-state-%D0%BF%D1%80%D0%BE%D1%82%D0%BE%D0%BA%D0%BE%D0%BB-%D0%BD%D0%B0-%D0%B2%D0%B0%D1%88%D0%B5-%D1%83%D1%81%D0%BC%D0%BE%D1%82%D1%80%D0%B5%D0%BD%D0%B8%D0%B5"> ссылка. </a> Только поставьте свои айпишники
Есть другой путь
Пишем на обеих RTR
```
firewall-cmd --permanent --zone=trusted -add-interface=gre1
firewall-cmd --reload
```
А дальше я хз как пароль сделать (по заданию
• Обеспечьте защиту выбранного протокола посредством парольной
защиты)

## Задание 8. Настройка динамической трансляции адресов маршрутизаторах HQ-RTR и BR-RTR
Мы это сделали в самом начале (Задание 2)
## Задание 9. Настройте протокол динамической конфигурации хостов для сети в сторону HQ-CLI
Скачиаем DHPC сервер
```
apt-get update && apt-get install dhcp-server
```
Далее настраиваем его
```
vim /etc/dhcp/dhcpd.conf

ddns-uptade-style none;
default-lease-time 600;
max-lease-time 7200;
authoritative;
log-facility local7;
subnet 192.168.200.0 netmask 255.255.255.240 {
range 192.168.200.2 192.168.200.14;
option routers 192.168.200.1;
option subnet-mask 255.255.255.240;
option domain-name-servers (Ваш ДНС сервер у ISP);
option domain-name "au-team.irpo";}
```
ЛИБО ЕСЛИ СВЕРХУ НЕ РАБОТАЕТ ПИШЕМ ЭТО
```
subnet 192.168.200.0 netmask 255.255.255.240 {
        option routers                  192.168.200.1;
        option subnet-mask              255.255.255.240;
        option domain-name              "au-team.irpo";
        option domain-name-servers      (Ваш ДНС сервер у ISP);
        range dynamic-bootp 192.168.200.66 192.168.200.78;
        default-lease-time 600;
        max-lease-time 7200;
}
```
Далее заходим в
```
vim /etc/sysconfig/dhcpd

DHCPDARGS="ens35" (ПИШИТЕ ИНТЕРФЕЙС, КОТОРЫЙ ПОДКЛЮЧЕН К HQ-CLI)
```
Запускаем HQ-CLI и настраиваем как в картинках (https://github.com/XBOSTICK1/demo2025?tab=readme-ov-file#включение-dhcp-в-hq-cli)
## Задание 10. Настройте инфраструктуру разрешения доменных имён для офисов HQ и BR:
Я хз как делать
## Задание 11. Настройте часовой пояс на всех устройствах (за исключением виртуального коммутатора, в случае его использования) согласно месту проведения экзамена
```
timedatectl set-timezone Asia/Yekaterinburg
```
Проверка
```
timedatectl
```
## Используемые источники

https://github.com/meowehh/DemoExam_2026
https://github.com/Elias888s/guutfg
