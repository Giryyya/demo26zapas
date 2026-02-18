# Информация по сети
### Схема сети:

 <details>
    <summary>НАЖМИ</summary>
<img width="541" height="532" alt="image" src="https://github.com/user-attachments/assets/e9735d52-ddad-4bfa-8791-efee06bacbbe" />

 </details>
  
### Таблица устройств:

| Название устройства | ОС | FQDN |
|:-|:-|:-|
| ISP (интерфейс в сторону HQ-RTR) | Alt Server | docker.au-team.irpo |
| ISP (интерфейс в сторону BR-RTR) | Alt Server | web.au-team.irpo |
| HQ-RTR  | EcoRouter/Alt | hq-rtr.au-team.irpo |
| BR-RTR | EcoRouter/Alt | br-rtr.au-team.irpo |
| HQ-SRV | Alt Server | hq-srv.au-team.irpo |
| BR-SRV | Alt Server | br-srv.au-team.irpo |
| HQ-CLI | Alt Workstation| hq-cli.au-team.irpo |

| Название устройства | IP-адрес |
|:-|:-|
| ISP - HQ-RTR | 172.16.1.1/28 |
| ISP - BR-RTR |  172.16.2.1/28|
| ISP - Internet | dhcp |
| HQ-RTR - ISP | 172.16.1.2/28 |
| BR-RTR - ISP | 172.16.2.2/28 |
|HQ-RTR - LAN | 192.168.1.1/26 |
|BR-RTR - LAN | 192.168.2.1/28 |
|HQ-SRV | 192.168.1.62/26 |
|BR-SRV | 192.168.2.14/28 |
| HQ-CLI | 192.168.1.3/26 |

# Модуль 1

## Произведите базовую настройку устройств
 <details>
    <summary>НАЖМИ</summary>
   
• Настройте имена устройств согласно топологии. Используйте полное 
доменное имя
```
cd /etc
vim hostname
```

### У Linux есть несколько виртуальных консолей, которые переключаются по Ctrl+Alt+F*.

Если не нужна графика можно отключить ее автозагрузку:
```
systemctl set-default multi-user.target
```
Для включения:
```
systemctl set-default graphical.target
```
Запустить графику можно через команду:
```
startx
```

  </details>

## На всех устройствах необходимо сконфигурировать IPv4

 <details>
    <summary>ЗАДАНИЕ</summary>
   
• IP-адрес должен быть из приватного диапазона, в случае, если сеть 
локальная, согласно RFC1918 
   
• Локальная сеть в сторону HQ-SRV(VLAN 100) должна вмещать не 
более 32 адресов 

• Локальная сеть в сторону HQ-CLI(VLAN 200) должна вмещать не 
менее 16 адресов 

• Локальная сеть для управления(VLAN 999) должна вмещать не 
более 8 адресов 

• Локальная сеть в сторону BR-SRV должна вмещать не более 16 
адресов 

• Сведения об адресах занесите в таблицу 2, в качестве примера 
используйте Прил_3_О1_КОД 09.02.06-3-2026-М1
 
 </details>

   <details>
    <summary>НАЖМИ</summary>

Для того чтобы посмотреть какие адаптеры подключены к устройству прописываем:

```
ip link show
```

Далее необходимо создать директорию по пути

```
mkdir /etc/net/ifaces/ не забываем про ivp4address(ip адрес) и ipv4route (шлюз)
```

Скопировать Options можно из ens33:

```
cp /etc/net/ifaces/ens33/options /etc/net/ifaces/ens37/options
```

Для выхода в интернет (ISP):

<img width="270" height="159" alt="image" src="https://github.com/user-attachments/assets/a8606f4f-b1dd-40bb-8e09-348c927a063c" />

Для локалки:

<img width="245" height="148" alt="image" src="https://github.com/user-attachments/assets/729d645d-c53c-4f88-bf1d-37cbf12e3596" />

ipv4address:

<img width="178" height="81" alt="image" src="https://github.com/user-attachments/assets/0614dc79-f4d1-4072-a885-0d84ad72a7f0" />

ipv4route(на HQ-RTR и BR-RTR тоже настраиваем):

<img width="223" height="27" alt="image" src="https://github.com/user-attachments/assets/6af8381c-3ec2-4a49-9efe-2d3cf6ac7eb3" />


Для того чтобы при перезапуске не сбрасывался адреса устройства необходимо в папке /etc/systemd/system создать файл сервиса:
1) network-restart.timer:
```
[Unit]
Description=Restart Network Timer

[Timer]
OnStartupSec=10s
Unit=network-restart.service

[Install]
WantedBy=timers.target
```
2) network-restart.service:
```
[Unit]
Description=Restart Network Service

[Service]
Type=oneshot
ExecStart=/bin/systemctl restart network

[Install]
WantedBy=multi-user.target
```
Чтобы запустить службу прописываем:
```
systemctl enable network-restart.timer
```
Прописываем днс на всех адаптера (лишним не будет) - файл /etc/net/ifaces/ens33/resolv.conf:
```
nameserver 8.8.8.8
```

  </details>

## Настройте доступ к сети Интернет, на маршрутизаторах ISP, HQ-RTR, BR-RTR

 <details>
    <summary>ЗАДАНИЕ</summary>
  
Настройте адресацию на интерфейсах: 
   
• Интерфейс, подключенный к магистральному провайдеру, получает 
адрес по DHCP 

• Настройте маршрут по умолчанию, если это необходимо 

• Настройте интерфейс, в сторону HQ-RTR, интерфейс подключен к сети 
172.16.1.0/28 

• Настройте интерфейс, в сторону BR-RTR, интерфейс подключен к сети 
172.16.2.0/28 

• На ISP настройте динамическую сетевую трансляцию портов для 
доступа к сети Интернет HQ-RTR и BR-RTR. 
 
 </details>

<details>
    <summary>НАЖМИ</summary> 
  
Откройте файл /etc/sysctl.conf и добавьте строку:
```
net.ipv4.ip_forward=1
```
Отредактируйте строчку в файле /etc/net/sysctl.conf:
```
net.ipv4.ip_forward=1
```
Пропишите команду для настройки динамической трансляции адресов (ens33 - смотрит в WAN, ens37 - LAN):
```
iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE
iptables -A FORWARD -i ens37 -o ens33 -j ACCEPT
iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sysctl -p
```
Далее необходимо сохранить настройки:
```
mkdir /etc/iptables
iptables-save>/etc/iptables/rules.v4
```
Для того чтобы после перезагрузки роутера не сбрасывались настройки необходимо прописать systemd-юнит iptables-restore.service:
```
[Unit]
Description=Restore iptables rules
Before=network.target

[Service]
Type=oneshot
ExecStart=/sbin/iptables-restore /etc/iptables/rules.v4

[Install]
WantedBy=multi-user.target
```
Далее необходимо включить юнит:
```
systemctl enable iptables-restore.service
systemctl start iptables-restore.service
```
Далее необходимо запустить iptables:
```
systemctl enable iptables
systemctl start iptables
```
Если не запустился, то вставляем в конфиг /etc/sysconfig/iptables минимальную конфигурацию и пытаемся опять запустить:
```
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
COMMIT
*nat
:PREROUTING ACCEPT [0:0]
:INPUT ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
COMMIT
```
Если iptables не запускается проверяем файл конфигурации на ошибки:
```
iptables-restore -t /etc/sysconfig/iptables
```
Если интернет на хостах не появился, еще раз прописываем:
```
sudo iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE
```

   </details> 

##  Создайте локальные учетные записи

 <details>
    <summary>ЗАДАНИЕ</summary>
  
 Создайте пользователя remote_user на HQ-SRV и BR-SRV
 
• Пароль пользователя sshuser с паролем P@ssw0rd 

• Идентификатор пользователя 2026 

• Пользователь sshuser должен иметь возможность запускать sudo без 
ввода пароля 

• Создайте пользователя net_admin на маршрутизаторах HQ-RTR и BR
RTR 

• Пароль пользователя net_admin с паролем P@ssw0rd 

• При настройке ОС на базе Linux, запускать sudo без ввода пароля 

• При настройке ОС отличных от Linux пользователь должен обладать 
максимальными привилегиями.

 </details>


  <details>
    <summary>НАЖМИ</summary>
  
Создаем пользователя remote_user с идентификатором 2026:
```
useradd -u 2026 remote_user
```

Идентификатор можно посмотреть с помощью команды id remote_user, либо в файле /etc/passwd

Задаем пароль:
```
passwd remote_user
```
Добавляем в группу sudo:
```
usermod -aG wheel remote_user
```
Для того чтобы при выполнении команды sudo не запрашивался пароль необходимо отредактировать файл /etc/sudoers. Вписываем:
```
remote_user ALL=(ALL) NOPASSWD: ALL
```

 </details>

 ##  Настройте коммутацию в сегменте HQ:

  <details>
    <summary>ЗАДАНИЕ</summary>
  
Трафик HQ-SRV должен принадлежать VLAN 100 

• Трафик HQ-CLI должен принадлежать VLAN 200 

• Предусмотреть возможность передачи трафика управления в VLAN 999 

• Реализовать на HQ-RTR маршрутизацию трафика всех указанных VLAN 
с использованием одного сетевого адаптера ВМ/физического порта 

• Сведения о настройке коммутации внесите в отчёт

 </details>

 <details>
    <summary>НАЖМИ</summary>
 
Для начала установим nmtui дабы удобнее было настраивать:
```
apt-get install NetworkManager-tui
```
По необходимости прописываем днс в /etc/net/ifaces/ens33/resolv.conf
```
nameserver 8.8.8.8
```

Запускаем nmtui:
```
systemctl start NetworkManager
systemctl status NetworkManager
```
Настройки для VLAN100 на HQ-RTR:

<img width="739" height="553" alt="image" src="https://github.com/user-attachments/assets/dd101b5b-d419-493f-8f4b-e8a0ea870f8b" />

На клиенте редачим файл /etc/net/interfaces:

```
auto ens33
iface ens33 inet manual
    up ip link set ens33 up

auto ens33.100
iface ens33.100 inet static
    address 192.168.1.30/27
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8
    vlan-raw-device ens33
```

В теории оно должно работать, но не работает днс

 </details>

 ## Настройте безопасный удаленный доступ на серверах HQ-SRV и BR-SRV

   <details>
    <summary>ЗАДАНИЕ</summary>
     
 Для подключения используйте порт 2026 
 
• Разрешите подключения исключительно пользователю sshuser 
• Ограничьте количество попыток входа до двух 

• Настройте баннер «Authorized access only». 

 </details>

   <details>
    <summary>НАЖМИ</summary>

Создаем пользователя для ssh:
```
useradd -m -s /bin/bash sshuser
passwd sshuser
```
Редактируем конфиг /etc/openssh/sshd_config(можно либо просто вписать эти строчки, либо найти и раскоментить их, в теории они все должны быть в конфиге):
```
Port 2026
AllowUsers sshuser
MaxAuthTries 2
Banner /etc/openssh/banner
PermitRootLogin no
Protocol 2
```
Создаем баннер по пути который указывали в конфиге: /etc/openssh/banner:
```
***********************************************
*         Authorized access only              *
*         VHOD TOL'KO DLYA KRUTIH             *
***********************************************
```
Прописываем правила для ssh в iptables на всех роутерах:
```
iptables -A INPUT -p tcp --dport 2026 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j DROP
sysctl -p
iptables-save>/etc/iptables/rules.v4
``` 
Проверяем конфиг на ошибки:
```
sshd -t
``` 
Если ошибок нет запускаем sshd:
```
systemctl restart sshd
```
Добавляем ssh на новый порт в автозагрузку SELinux (для альта обычно не нужно, но на всякий пропишем):
```
semanage port -a -t ssh_port_t -p tcp 2026 2>/dev/null || true
```
Подключаемся:
```
ssh -p 2026 sshuser@192.168.1.62
```

 </details>

 ## Между офисами HQ и BR, на маршрутизаторах HQ-RTR и BR-RTR необходимо сконфигурировать ip туннель

   <details>
    <summary>ЗАДАНИЕ</summary>
  
    На выбор технологии GRE или IP in IP 
    
• Сведения о туннеле занесите в отчёт.

 </details>

   <details>
    <summary>НАЖМИ</summary>
  
 Создаем директорию для туннеля:
 ```
mkdir /etc/net/ifaces/tun1
```
Редактируем файл options следующим образом (HQ-RTR):

<img width="191" height="196" alt="image" src="https://github.com/user-attachments/assets/107f46e5-c8c4-43b8-84fb-ae5e7c9a8ad3" />

Здесь TUNLOCAL - IP адресс адаптера в сторону ISP на настраиваемом роутере, TUNREMOTE на другом роутере.

Задаем IP адрес:
```
echo 10.10.10.1/30 > /etc/net/ifaces/tun1/ipv4address
```
Перезагружаем сеть:
```
systemctl restart network
```

 </details>

 ##  Обеспечьте динамическую маршрутизацию на маршрутизаторах HQ-RTR и BR-RTR

  <details>
    <summary>ЗАДАНИЕ</summary>
 
 сети одного офиса должны быть доступны из другого 
офиса и наоборот. Для обеспечения динамической маршрутизации 
используйте link state протокол на усмотрение участника:

• Разрешите выбранный протокол только на интерфейсах ip туннеля 

• Маршрутизаторы должны делиться маршрутами только друг с другом 

• Обеспечьте защиту выбранного протокола посредством парольной 
защиты 

• Сведения о настройке и защите протокола занесите в отчёт.

 </details>

<details>
    <summary>НАЖМИ</summary>
 
Устанавливаем frr:
```
apt-get install frr -y
```

Включаем OSPF в конфиге /etc/frr/daemons:

<img width="863" height="684" alt="image" src="https://github.com/user-attachments/assets/889ab48a-5ad0-45ea-ae7e-e93045da4910" />

Запускаем frr:
```
systemctl enable --now frr
```
Переходим в консоль frr:
```
vtysh
```
Настраиваем настройки в терминале:
```
conf t
router ospf
passive-interface default
network 192.168.1.0/26 area 0
network 192.168.2.0/28 area 0
network 10.10.10.0/30 area 0
area 0 authentication
exit
```
Настраиваем интерфейсы(туннель)(все там же):
```
interface tun1
no ip ospf network broadcast
no ip ospf passive
ip ospf authentication
ip ospf authentication-key password
exi:
```
Не забываем сохранить изменения (как в циске):
```
write
```
Перезапускаем frr:
```
systemctl restart frr
```
Проверяем конфиг /etc/frr/frr.conf:

<img width="995" height="654" alt="image" src="https://github.com/user-attachments/assets/2b8595f1-2102-46b9-bea6-acf70527605a" />

Проверить можно либо пингом либо:
```
vtysh
show ip ospf route
```

<img width="498" height="220" alt="image" src="https://github.com/user-attachments/assets/f7df5030-7a98-4561-a69c-ba35601ba708" />

 </details>

##  Настройка динамической трансляции адресов маршрутизаторах HQ-RTR и BR-RTR

<details>
    <summary>НАЖМИ</summary>

### Делали раннее, если не делали, то настройки как на ISP

 </details>

 ## Настройте протокол динамической конфигурации хостов для сети в сторону HQ-CLI

 <details>
    <summary>ЗАДАНИЕ</summary>

Настройте нужную подсеть 

• В качестве сервера DHCP выступает маршрутизатор HQ-RTR 
• Клиентом является машина HQ-CLI

• Исключите из выдачи адрес маршрутизатора 

• Адрес шлюза по умолчанию – адрес маршрутизатора HQ-RTR 

• Адрес DNS-сервера для машины HQ-CLI – адрес сервера HQ-SRV 

• DNS-суффикс – au-team.irpo 

• Сведения о настройке протокола занесите в отчёт.

 </details>

 <details>
    <summary>НАЖМИ</summary>
  
  Для начала укажем сетевой интерфейс, через который будет работать DHCP-сервер:
```
vim /etc/sysconfig/dhcpd
```

<img width="567" height="147" alt="image" src="https://github.com/user-attachments/assets/cd9ea117-b1f3-43e2-afa7-785c9851fbd5" />

В папке /etc/dhcp/ необходимо создать файл dhcpd.conf:
```
cp dhcpd.conf.example dhcpd.conf
```
Отредактируйте файл dhcpd.conf следующим образом:

<img width="564" height="204" alt="image" src="https://github.com/user-attachments/assets/e7392380-c2cd-48a2-8484-c4701e7ce6ca" />

Перезагружаем службу:
```
systemctl restart dhcpd
```

Чтобы служба включалась после перезапуска устройства можно добавить ее в systemd юнит следующим образом(редактируется служба network-restart.service):

![image](https://github.com/user-attachments/assets/e929cbfd-2d7e-49c1-9a27-db636dfb165c)

 </details>

## Настройте инфраструктуру разрешения доменных имён для офисов HQ и BR

<details>
    <summary>ЗАДАНИЕ</summary>

Основной DNS-сервер реализован на HQ-SRV 

• Сервер должен обеспечивать разрешение имён в сетевые адреса 
устройств и обратно в соответствии с таблицей 3 

• В качестве DNS сервера пересылки используйте любой общедоступный 
DNS сервер(77.88.8.7, 77.88.8.3 или другие)

 </details>

 <details>
    <summary>НАЖМИ</summary>

Для начала необходимо отредактировать файл /etc/bind/options.conf:
```
listen-on { any; };
allow-query { any; };
allow-transfer { 192.168.33.254; };
```
```
systemctl restart network
```
Автозагрузка bind:
```
systemctl enable --now bind
```
Создаем прямую и обратную зону в /etc/bind/local.conf:

<img width="902" height="683" alt="image" src="https://github.com/user-attachments/assets/ba19e661-36c9-4a0a-aa3e-53a43b503f2a" />

Копируем дефолты:
```
cp /etc/bind/zone/{localhost,au.db}
cp /etc/bind/zone/127.in-addr.arpa /etc/bind/zone/1.168.192.in-addr.arpa.db
cp /etc/bind/zone/127.in-addr.arpa /etc/bind/zone/2.168.192.in-addr.arpa.db
```
Назначаем права:
```
chown root:named /etc/bind/zone/au.db
chown root:named /etc/bind/zone/1.168.192.in-addr.arpa.db
chown root:named /etc/bind/zone/2.168.192.in-addr.arpa.db
```
Настраиваем зону прямого просмотра /etc/bind/zone/au.db:

<img width="897" height="657" alt="image" src="https://github.com/user-attachments/assets/86b0b940-e5f3-4000-b5e1-cc2e63ac3c3e" />

Настраиваем зону обратного просмотра /etc/bind/zone/1.168.192.in-addr.arpa.db:

<img width="918" height="681" alt="image" src="https://github.com/user-attachments/assets/326fc7d8-10f9-48af-ab0d-0c1f0c8c94a5" />

Настраиваем зону обратного просмотра /etc/bind/zone/2.168.192.in-addr.arpa.db:

<img width="883" height="716" alt="image" src="https://github.com/user-attachments/assets/70c19ee1-57a3-4c7e-9d83-bc942ac153a1" />

Проверяем зоны:
```
named-checkconf -z
```

<img width="506" height="189" alt="image" src="https://github.com/user-attachments/assets/fde35c82-e213-4e02-86ed-5d72d87e1ffb" />

В файле /etc/hosts прописываем адрес сервера:
```
192.168.1.62   hq-srv.au-team.irpo hq-srv
```

<img width="863" height="690" alt="image" src="https://github.com/user-attachments/assets/edb3b0c9-a488-42db-9857-be7dfe9e338c" />

Чтобы днс заработал на клиентах необходимо добавить адрес сервера в resolv.conf. Это делается разными способами и зависит по всей видимости от типа ОС (сервер/воркстейшн) и (почему-то) от наличия графики. Конечной целью будет привести файл /etc/resolv.conf к следующему виду:

<img width="450" height="123" alt="image" src="https://github.com/user-attachments/assets/328af76d-079d-411f-ab39-c770fbfdce94" />

Это можно сделать разными способами:

### 1 способ:

В /etc/net/ifaces/ens33/resolv.conf прописываем:
```
search au.team
nameserver 192.168.1.62
```
И так делаем во всех интерфейсах (и на днс сервере тоже делаем)

Перезагружаем сеть:
```
systemctl restart network
```
Если адрес сервера не появился то просто удаляем /etc/resolv.conf и прописываем туда тоже самое что и в resolv.conf на интерфейсах:
```
rm resolv.conf
rm resolv.conf~
```


### 2 способ (чаще работает):

В /etc может лежать файлик resolvconf.conf (как показывает практика работает он на ОС без графики). В нем указываем следующее:

<img width="1006" height="702" alt="image" src="https://github.com/user-attachments/assets/ab594bf8-3399-45b1-b5ad-89537df1cdf7" />


 </details>
