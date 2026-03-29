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

 ## ПОД РУТА ПРОВАЛИВАЕМСЯ ТОЛЬКО КОМАНДОЙ sudo -i ИЛИ sudo su - (это аналог команды, только в виде связки команд), ЕСЛИ ОНА НЕ РАБОТАЕТ, ТО ЗАХОДИМ ПОД НЕГО ЧЕРЕЗ tty2 (ctrl+alt+f2)
 
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
локальная, согласно RFC1918 (10.0.0.0/8, 192.168.0.0/16, 172.16.0.0/12)
   
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
exit
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

###  ВАЖНО!!! Я добавил в зону прямого просмотра все устройства кроме ISP и сделал обратные зоны для всех устройств в подсетях 1.0 и 2.0. Смотрите задание, там могут быть другие условия.
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

##  Настройте часовой пояс на всех устройствах согласно месту проведения экзамена

<details>
    <summary>НАЖМИ</summary>
 
Задавать часовой пояс будем с помощью утилию timedatetcl:
```
timedatectl set-timezone Europe/Moscow
timedatectl
```

Сами зоны расположены по пути /usr/share/zoneinfo/

 </details>

# Модуль 2

## Настройте контроллер домена Samba DC на сервере BR-SRV

<details>
    <summary>ЗАДАНИЕ</summary>
 
Имя домена au-team.irpo 

• Введите в созданный домен машину HQ-CLI

• Создайте 5 пользователей для офиса HQ: имена пользователей формата 
hquser№ (например hquser1, hquser2 и т.д.) 

• Создайте группу hq, введите в группу созданных пользователей 

• Убедитесь, что пользователи группы hq имеют право 
аутентифицироваться на HQ-CLI 

• Пользователи группы hq должны иметь возможность повышать 
привилегии для выполнения ограниченного набора команд: cat, grep, id. 
Запускать другие команды с повышенными привилегиями пользователи 
группы права не имеют.

 </details>

<details>
    <summary>НАЖМИ</summary>
 
### Не факт что работает и по хорошему при установке альта поставить все что связано с доменами, у меня только там смогло зайти в юзера

Устанавливаем необходимые пакеты:
```
apt update
apt install samba-dc samba-winbind krb5-kdc chrony iptables
```
Редактируем хрони /etc/chrony.conf:
```
server pool.ntp.org iburst
```
Перезапускаем хрони:
```
systemctl enable --now chronyd
```
Останавливаем стандартные службы самбы, если запущены:
```
systemctl stop smbd nmbd winbind
systemctl disable smbd nmbd winbind
```
Создаем домен:
```
rm /etc/samba/smb.conf
samba-tool domain provision --use-rfc2307 --interactive
```
```
Realm: AU.TEAM
Domain: AU
Server Role: dc
DNS backend: SAMBA_INTERNAL 
DNS forwarder: 192.168.1.62 
Administrator password: P@ssw0rd 
```
Запускаем самбу (сначала нужно ребутнуть сервер, иначе будут ошибки):
```
systemctl enable --now samba
```
Открываем порты в iptables (на роутерах):
```
iptables -A INPUT -p tcp -m multiport --dports 53,88,135,139,389,445,464,636 -j ACCEPT
iptables -A INPUT -p udp -m multiport --dports 53,88,123,137,138,389,464 -j ACCEPT
iptables-save > /etc/iptables/rules.v4
```
Проверяем домен на работоспособность:
```
samba-tool domain level show
samba-tool user list
```

<img width="488" height="280" alt="image" src="https://github.com/user-attachments/assets/5867b327-4629-4271-8289-c4c75df2a0a2" />

Создаем группу hq:
```
samba-tool group add hq
```
Создаеми юзеров:
```
for i in {1..5}; do
  samba-tool user create hquser$i 'P@ssw0rd'
  samba-tool group addmembers hq hquser$i
done
```
Проверяем группу:
```
samba-tool group listmembers hq
```

<img width="428" height="111" alt="image" src="https://github.com/user-attachments/assets/8d022e52-ef35-4bc3-b473-9dc2cd3ba69a" />

### Далее необходимо завести машины в домен:

Проверяем /etc/resolv.conf:
```
search au.team
nameserver 192.168.2.14
nameserver 192.168.1.62
```
Проверяем доступность:
```
ping au.team
nslookup _ldap._tcp.au.team
```
Устанавливаем пакеты (Если при установке выбрали все что связано с доменами, то не нужно, если нет графики, то нужно):
```
apt-get install realmd sssd sssd-tools adcli krb5-workstation samba-common-tools
```

Заходим в центр управления системой:

<img width="724" height="475" alt="image" src="https://github.com/user-attachments/assets/585bda4f-6b83-4850-88ad-6258d6c5913c" />

Ищем кнопку аутентификация:

<img width="1233" height="704" alt="image" src="https://github.com/user-attachments/assets/81c96671-a630-4ab7-a3c5-52997d9acb4c" />

Вводим все как на скрине в Active Directory и жмем кнопку применить снизу, система попросит ввести учетку админа домена

<img width="1284" height="739" alt="image" src="https://github.com/user-attachments/assets/e9de9b3a-3e0f-4586-90a8-813e04cb8595" />

### на устройствах без графики:

После установки пакетов обнаруживаем домен:
```
realm discover au.team
```
Заходим в домен:
```
realm join -U Administrator au.team
```
### Настраиваем группу hq на hq-cli:
Задаем права на sudo:
```
chown root:root /usr/bin/sudo
chmod 4755 /usr/bin/sudo
ls -l /usr/bin/sudo
```
Вывод должен быть следующий:

<img width="487" height="44" alt="image" src="https://github.com/user-attachments/assets/26a66c8c-591e-47c4-bc9d-9f31fa7c308b" />

!!!Нужно обратить вниманию на букву s на 4 позиции!!!

Проверяем пути к командам:
```
which cat grep id
```
Редактируем /etc/sudoers:
```
%hq ALL=(ALL) /usr/bin/cat, /bin/grep, /usr/bin/id
```
Заходим под hquser1:
```
su - hquser1
```
Проверяем команды:
```
sudo cat /etc/hostname
sudo grep root /etc/passwd
sudo id
```
Если не работает, то в /etc/sudoers прописываем другие настройки (предыдущую строчку удалить):
```
Cmnd_Alias HQ_COMMANDS = /bin/cat, /bin/grep, /usr/bin/id
%hq ALL=(ALL) HQ_COMMANDS
```

 </details>

## Сконфигурируйте файловое хранилище на сервере HQ-SRV

<details>
    <summary>ЗАДАНИЕ</summary>
 
При помощи двух подключенных к серверу дополнительных дисков 
размером 1 Гб сконфигурируйте дисковый массив уровня 0 

• Имя устройства – md0, при необходимости конфигурация массива 
размещается в файле /etc/mdadm.conf 

• Создайте раздел, отформатируйте раздел, в качестве файловой системы 
используйте ext4 

• Обеспечьте автоматическое монтирование в папку /raid

 </details>

 <details>
    <summary>НАЖМИ</summary>
 
Подключаем диски через Vmware:

<img width="342" height="310" alt="image" src="https://github.com/user-attachments/assets/36c77e6f-324c-475c-8cfd-dc4909c79dca" />

Устанавливаем mdadm:
```
apt-get install mdadm -y
```
Смотрим какие диски у нас имеются:
```
lsblk
```

<img width="619" height="147" alt="image" src="https://github.com/user-attachments/assets/c927e347-a2a6-41cf-a2ef-5d9dea7bb214" />

Далее создаем разделы у дисков:
```
cfdisk /dev/sdb
```

Выбираем gpt:

<img width="757" height="520" alt="image" src="https://github.com/user-attachments/assets/e6fda1aa-29c7-4b1c-b612-1fd0b35ef848" />

Жмем кнопку New:

<img width="1069" height="744" alt="image" src="https://github.com/user-attachments/assets/6d22b1a9-4e9a-4cb6-b68a-d3769d30f8e8" />

Выбираем максимальный размер:

<img width="1097" height="741" alt="image" src="https://github.com/user-attachments/assets/138f761c-3aab-4471-9556-db7347c3e6c9" />

Записываем изменения (рулить в утилите на стрелочки):

<img width="1096" height="750" alt="image" src="https://github.com/user-attachments/assets/5953107b-1436-4179-b8f1-bfc79883e76f" />

Полностью прописываем yes:

<img width="877" height="276" alt="image" src="https://github.com/user-attachments/assets/d7c400f2-fb05-4cf7-a307-fda30e30a49a" />

По итогу должно получиться так:

<img width="633" height="201" alt="image" src="https://github.com/user-attachments/assets/7ae1694f-065c-479d-92cc-def149e2df1b" />
 
Создаем массив md0 0 уровня:
```
mdadm --create --verbose /dev/md0 -l 0 -n 2 /dev/sdb1 /dev/sdc1
```

Проверяем:

<img width="656" height="213" alt="image" src="https://github.com/user-attachments/assets/b1c550d2-5b0b-43a1-8466-aaa8559a87eb" />

Создаем таблицу раздела:
```
mkfs -t ext4 /dev/md0
```
Сохраняем конфигураци:
```
mdadm --detail --scan --verbose /dev/md0 > /etc/mdadm.conf
```
Монтируем массив:
```
echo '/dev/md0 /raid ext4 defaults 0 0' >> /etc/fstab
mount /dev/md0 /raid
```
 </details>

 ## Настройте сервер сетевой файловой системы (nfs) на HQ-SRV

 <details>
    <summary>ЗАДАНИЕ</summary>
 
В качестве папки общего доступа выберите /raid/nfs, доступ для чтения и записи исключительно для сети в сторону HQ-CLI 

• На HQ-CLI настройте автомонтирование в папку /mnt/nfs 

• Основные параметры сервера отметьте в отчёте

 </details>

  <details>
    <summary>НАЖМИ</summary>

### На сервере: 
Создаем папку в /raid:
```
mkdir /raid/nfs
```
Скачиваем nfs:
```
apt-get install nfs-utils
```
В файле /etc/exports добавляем строчку:
```
/raid/nfs 192.168.1.0/26(rw,sync,no_root_squash,subtree_check)
```
Запускаем nfs:
```
systemctl enable --now nfs-server
```
Проверяем появилась ли папка:
```
exportfs
```

<img width="271" height="62" alt="image" src="https://github.com/user-attachments/assets/491f2649-93c5-4de1-91ed-91636d116fbd" />

Если не появилась - еще раз ребутаем nfs.

### На клиенте:
Устанавливаем nfs:
```
nfs-utils
```
Создаем папку:
```
mkdir /mnt/nfs
```
Монтируем папку:
```
mount -t nfs 192.168.1.62:/raid/nfs /mnt/nfs
```
Настраиваем автоматическое монтирование:
```
echo '192.168.1.62:/raid/nfs /mnt/nfs nfs auto 0 0 ' >> /etc/fstab
```

 </details>

 ## Настройте службу сетевого времени на базе сервиса chrony на маршрутизаторе ISP

 <details>
    <summary>ЗАДАНИЕ</summary>
 
Вышестоящий сервер ntp на маршрутизаторе ISP - на выбор участника 

• Стратум сервера - 5 

• В качестве клиентов ntp настройте: HQ-SRV, HQ-CLI, BR-RTR, BR-SRV.

 </details>

 <details>
    <summary>НАЖМИ</summary>
 
Настраиваем маршрутизацию до ISP:
### ISP:
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
network 172.16.1.0/28 area 0
network 172.16.2.0/28 area 0
area 0 authentication
exit
```
Настраиваем интерфейсы(ens37, ens38):
```
interface ens37
no ip ospf network broadcast
no ip ospf passive
ip ospf authentication
ip ospf authentication-key password
exit
```
Не забываем сохранить изменения (как в циске):
```
write
```
Перезапускаем frr:
```
systemctl restart frr
```
### HQ-RTR и BR-RTR:

В frr необходимо добавить сети ISP:
```
vtysh
conf t
router ospf
network 172.16.1.0/28 area 0
network 172.16.2.0/28 area 0
ex
interface ens33
no ip ospf network broadcast
no ip ospf passive
ip ospf authentication
ip ospf authentication-key password
exit
do wr
```
Перезапускаем frr:
```
systemctl restart frr
```
### Настраиваем chrony на ISP:

Устанавливаем Chrony:
```
apt-get install chrony -y
```
Редактируем Файл /etc/chrony.conf:
```
server 127.0.0.1 iburst prefer
local stratum 5
allow 192.168.1.0/26
allow 192.168.2.0/28
```
Должно выглядеть так:

<img width="852" height="228" alt="image" src="https://github.com/user-attachments/assets/14fe6b26-2773-47d9-afcd-d3155e7efc76" />

Перезазгружаем Chrony:
```
systemctl restart chronyd
```
### На клиенте:
Устанавливаем Chrony:
```
apt-get install chrony -y
```
Редачим /etc/chrony.conf (192.168.189.131 - IP на ISP который смотрит в сеть):
```
server 192.168.189.131 iburst
```
Перезапускаем Chrony:
```
systemctl restart chronyd
```
Проверяем:
```
chronyc sources
```

Если стратум не 5, то можно попробовать поставить ntp вместо chrony:
```
apt-get install ntp
vim /etc/ntp.conf
```
Указываем:
```
server 127.127.1.0
server ntp2.vniiftri.ru iburst
fudge 127.127.1.0 stratum 5
restrict 192.168.11.0 mask 255.255.255.0 nomodify notrap
restrict 192.168.33.0 mask 255.255.255.0 nomodify notrap
```
Запускаем и проверяем:
```
systemctl enable ntp
systemctl start ntp
ntpq -p
```
 </details>

## Сконфигурируйте ansible на сервере BR-SRV

 <details>
    <summary>ЗАДАНИЕ</summary>
 
Сформируйте файл инвентаря, в инвентарь должны входить HQ-SRV, HQ-CLI, HQ-RTR и BR-RTR 

• Рабочий каталог ansible должен располагаться в /etc/ansible 

• Все указанные машины должны без предупреждений и ошибок отвечать 
pong на команду ping в ansible посланную с BR-SRV.

 </details>

  <details>
    <summary>НАЖМИ</summary>
 
Устанавливаем Ansible:
```
apt-get install ansible -y
```
Заходим под sshuser и генерируем ключи, которые будут расположены в /home/sshuser/.ssh:
```
ssh -p 2026 sshuser@192.168.2.14
ssh-keygen
```
Отправляем открытый ключ на другие хосты (Для начала необходимо настроить ssh где не настроен) (Смотрите на порты, у меня по итогу не оч заработало если везде 2026):
```
ssh-copy-id -p 2026 sshuser@192.168.1.62
ssh-copy-id -p 2026 sshuser@192.168.1.3
ssh-copy-id -p 2026 sshuser@192.168.2.1
ssh-copy-id -p 2026 sshuser@192.168.1.1
```
Создаем файл инвентаря и вносим туда хосты /etc/ansible/hosts:
```
[hq]
192.168.1.1 ansible_port=2026 ansible_user=sshuser ansible_python_interpreter=/usr/bin/python3 ansible_connection=local
192.168.1.3 ansible_port=2026 ansible_user=sshuser ansible_python_interpreter=/usr/bin/python3
192.168.1.62 ansible_port=2026 ansible_user=sshuser ansible_python_interpreter=/usr/bin/python3

[br]
192.168.2.1 ansible_port=2026 ansible_user=sshuser ansible_python_interpreter=/usr/bin/python3
```
### Если будут возникать ошибки формата как на скрине, то каждому хосту прописываем параметр ansible_connection=local:

<img width="1703" height="525" alt="image" src="https://github.com/user-attachments/assets/6ca8ad75-4aa3-4d79-951a-e93fb83f14c0" />

Редактируем конфиг /etc/ansible/ansible.cfg:
```
[defaults]
interpreter_python=auto_silent
log_path = /home/sshuser/ansible.log
```
Проверяем работоспособность:
```
ansible all -m ping
```

 </details>

## Разверните веб приложение в docker на сервере BR-SRV

 <details>
    <summary>ЗАДАНИЕ</summary>
 
Средствами docker должен создаваться стек контейнеров с веб приложением и базой данных 

• Используйте образы site_latestи mariadb_latestрасполагающиеся в директории docker в образе Additional.iso 

• Основной контейнер testapp должен называться tespapp 

• Контейнер с базой данных должен называться db 

• Импортируйте образы в docker, укажите в yaml файле параметры подключения к СУБД, имя БД - testdb, пользователь testс паролем P@ssw0rd, порт приложения 8080, при необходимости другие параметры 

• Приложение должно быть доступно для внешних подключений через порт 8080

 </details>

 <details>
    <summary>НАЖМИ</summary>
 
Устанавливаем Docker:
```
apt-get install docker-ce docker-compose -y
```
Запускаем Docker:
```
systemctl enable --now docker
```
Прооверяем работоспособность:
```
docker run hello-world
```
Создаем директорию для монтирования образа:
```
mkdir -p /mnt/iso
```
Создаем site_latest:
```
mkdir -p ~/docker-site
cd ~/docker-site
```
Создаем файл index.php и прописываем в нем:
```
<?php
echo "<h1>Test Application</h1>";
echo "<h2>Connection to Database:</h2>";

$host = getenv('DB_HOST') ?: 'db';
$dbname = getenv('DB_NAME') ?: 'testdb';
$user = getenv('DB_USER') ?: 'test';
$pass = getenv('DB_PASSWORD') ?: 'P@ssw0rd';

try {
    $pdo = new PDO("mysql:host=$host;dbname=$dbname;charset=utf8", $user, $pass);
    echo "<p style='color: green;'>Connected to MariaDB successfully!</p>";
    
    // Create test table
    $pdo->exec("CREATE TABLE IF NOT EXISTS test (id INT AUTO_INCREMENT PRIMARY KEY, message VARCHAR(255))");
    echo "<p>Test table created/checked</p>";
    
    // Insert test data
    $stmt = $pdo->prepare("INSERT INTO test (message) VALUES (?)");
    $stmt->execute(['Hello from Docker!']);
    echo "<p>Test data inserted</p>";
    
    // Read test data
    $result = $pdo->query("SELECT * FROM test");
    echo "<h3>Database entries:</h3><ul>";
    while ($row = $result->fetch()) {
        echo "<li>" . htmlspecialchars($row['message']) . "</li>";
    }
    echo "</ul>";
    
} catch (PDOException $e) {
    echo "<p style='color: red;'>Connection failed: " . $e->getMessage() . "</p>";
}
echo "<p>App port: " . (getenv('APP_PORT') ?: '8080') . "</p>";
phpinfo();
?>
```
Создаем Dockerfile и прописываем в нем:
```
FROM php:7.4-apache

# Install PHP extensions for MariaDB/MySQL
RUN docker-php-ext-install pdo_mysql mysqli

# Copy application files
COPY index.php /var/www/html/
COPY info.php /var/www/html/

# Set working directory
WORKDIR /var/www/html

# Configure Apache
RUN a2enmod rewrite

# Expose port
EXPOSE 8080

# Modify apache to use port 8080
RUN sed -i 's/80/8080/g' /etc/apache2/sites-available/000-default.conf
RUN sed -i 's/80/8080/g' /etc/apache2/ports.conf

# Start Apache
CMD ["apache2-foreground"]
```
Создаем info.php и прописываем в нем:
```
<?php
phpinfo();
?>
```
Собираем образ веб-приложения:
```
docker build -t site_latest .
```
Скачиваем MariaDB:
```
docker pull mariadb:10.5
```
Создаем образ:
```
docker tag mariadb:10.5 mariadb_latest
```
Создаем docker-compose.yml командой:
```
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  db:
    image: mariadb_latest
    container_name: db
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: testdb
      MYSQL_USER: test
      MYSQL_PASSWORD: P@ssw0rd
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - app_network
    ports:
      - "3306:3306"

  testapp:
    image: site_latest
    container_name: tespapp
    restart: unless-stopped
    depends_on:
      - db
    environment:
      DB_HOST: db
      DB_PORT: 3306
      DB_NAME: testdb
      DB_USER: test
      DB_PASSWORD: P@ssw0rd
      APP_PORT: 8080
    ports:
      - "8080:8080"
    networks:
      - app_network

networks:
  app_network:
    driver: bridge

volumes:
  db_data:
EOF
```
Проверяем что образы созданы:
```
docker images | grep -E "site_latest|mariadb_latest"
```
Проверяем конфиг (должен вывестись сам конфиг):
```
docker compose config
```
Запускаем контейнеры:
```
docker compose up -d
docker compose up -d testapp
```
Если ловим такую ошибку, то делаем следующее:

<img width="1704" height="133" alt="image" src="https://github.com/user-attachments/assets/c45fcc5d-cebd-44d7-8827-871d57d0e686" />

Смотрим чем занят порт и убиваем процесс (и желательно выключить службу которая занимает порт, иначе при перезагрузке ничего не заработает):
```
netstat -tulpn | grep 8080
ss -tulpn | grep 8080
lsof -i :8080
kill -9 2847
```

<img width="826" height="52" alt="image" src="https://github.com/user-attachments/assets/366965e8-5d81-459f-a2db-b8ba92552d27" />

Удаляем контейнер:
```
docker rm -f tespapp
```
Останавливаем контейнеры:
```
docker compose down
```
Проверяем порт еще раз:
```
ss -tulpn | grep 8080
```
Запускаем заново и смотрим:
```
docker compose up -d
docker compose ps
```

<img width="1236" height="200" alt="image" src="https://github.com/user-attachments/assets/49ab6451-9061-4f77-bcfc-7786362be832" />

Проверяем работоспособность:

<img width="1716" height="917" alt="image" src="https://github.com/user-attachments/assets/daf0a120-40ed-4aa6-b350-4f375d9f621c" />

Для того чтобы другие хосты увидели данное творение необходимо настроить iptables на BR-SRV:
```
iptables -F
iptables -X
iptables -t nat -F
iptables -t nat -X
iptables -t mangle -F
iptables -t mangle -X
iptables -P INPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -P OUTPUT ACCEPT
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -p icmp -j ACCEPT
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
iptables -A INPUT -p tcp --dport 8080 -j ACCEPT
iptables -A INPUT -p tcp --dport 3306 -j ACCEPT
iptables-save > /etc/sysconfig/iptables
mkdir /etc/iptables
iptables-save > /etc/iptables/rules.v4
iptables -L -n -v
```
Перезапускаем Docker и проверяем правила:
```
systemctl restart docker
iptables -L DOCKER -n -v
iptables -t nat -L DOCKER -n -v
iptables -L INPUT -n -v | grep -E "8080|8081"
```

<img width="1081" height="249" alt="image" src="https://github.com/user-attachments/assets/204b1b80-3a80-42c7-ab2f-993374c0d54b" />

### Для теста я поднимал еще один контейнер на 8081 порту, у вас его быть не должно и можно даже правило для него не создавать и не проверять ничего для этого порта

<img width="861" height="71" alt="image" src="https://github.com/user-attachments/assets/f97bdfb7-a4bd-4ae7-8292-765a7263d787" />

Перезапускаем контейнер:
```
docker compose down
docker compose up -d
```
Проверяем на других хостах (либо через браузер http://192.168.2.14:8080):
```
curl -v http://192.168.2.14:8080
```

 </details> 

## Разверните веб приложениена сервере HQ-SRV

 <details>
    <summary>ЗАДАНИЕ</summary>
 
Используйте веб-сервер apache 

• В качестве системы управления базами данных используйте mariadb 

• Файлы веб приложения и дамп базы данных находятся в директории web образа Additional.iso 

• Выполните импорт схемы и данных из файла dump.sql в базу данных webdb 

• Создайте пользователя webс паролем P@ssw0rd и предоставьте ему права доступа к этой базе данных 

• Файлы index.php и директорию images скопируйте в каталог веб сервера apache 

• В файле index.php укажите правильные учётные данные для подключения к БД 

• Запустите веб сервер и убедитесь в работоспособности приложения 

• Основные параметры отметьте в отчёте

 </details>

 <details>
    <summary>НАЖМИ</summary>

 ### Если есть ISO файл:

 Создаем директорию для монтирования:
```
mkdir -p /mnt/iso
```
Монтируем ISO-образ (нужно знать путь к Additional.iso):
```
mount -o loop /путь/к/Additional.iso /mnt/iso
```
Копируем директорию web:
```
cp -r /mnt/iso/web ~/
```
Проверяем содержимое:
```
ls -la ~/web/
```
Размонтируем ISO:
```
umount /mnt/iso
```
### Если ISO нет:
Создаем директорию:
```
mkdir -p ~/web
cd ~/web
```
Создаем dump.sql (дамп базы данных) командой:
```
cat > ~/web/dump.sql << 'EOF'
CREATE DATABASE IF NOT EXISTS webdb;
USE webdb;

CREATE TABLE IF NOT EXISTS users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS products (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    price DECIMAL(10,2),
    description TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS test (
    id INT AUTO_INCREMENT PRIMARY KEY,
    message VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO users (username, email) VALUES
    ('admin', 'admin@example.com'),
    ('user1', 'user1@example.com'),
    ('user2', 'user2@example.com');

INSERT INTO products (name, price, description) VALUES
    ('Product 1', 19.99, 'First test product'),
    ('Product 2', 29.99, 'Second test product'),
    ('Product 3', 39.99, 'Third test product');

INSERT INTO test (message) VALUES
    ('Hello from HQ-SRV!'),
    ('Database connection successful'),
    ('Test entry 3');
EOF
```
Создание Index.php(главная страница приложения) командой:
```
cat > ~/web/index.php << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>Web Application on HQ-SRV</title>
    <style>
        body { font-family: Arial; margin: 40px; background: #f5f5f5; }
        .container { max-width: 1200px; margin: 0 auto; background: white; padding: 20px; border-radius: 10px; box-shadow: 0 0 10px rgba(0,0,0,0.1); }
        h1 { color: #333; border-bottom: 2px solid #4CAF50; padding-bottom: 10px; }
        .success { color: #4CAF50; font-weight: bold; }
        .error { color: #f44336; font-weight: bold; }
        table { border-collapse: collapse; width: 100%; margin: 20px 0; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
        th { background: #4CAF50; color: white; }
        img { max-width: 200px; margin: 10px; border: 1px solid #ddd; border-radius: 5px; }
        .info { background: #e3f2fd; padding: 10px; border-radius: 5px; margin: 10px 0; }
    </style>
</head>
<body>
    <div class="container">
        <h1>Web Application on HQ-SRV (192.168.1.62)</h1>
        
        <div class="info">
            <h3>Server Information:</h3>
            <p><strong>Hostname:</strong> <?php echo gethostname(); ?></p>
            <p><strong>Server IP:</strong> 192.168.1.62</p>
            <p><strong>Date:</strong> <?php echo date('Y-m-d H:i:s'); ?></p>
            <p><strong>PHP Version:</strong> <?php echo phpversion(); ?></p>
        </div>
        
        <h2>Database Connection</h2>
        <?php
        $host = 'localhost';
        $dbname = 'webdb';
        $user = 'web';
        $pass = 'P@ssw0rd';
        
        try {
            $pdo = new PDO("mysql:host=$host;dbname=$dbname;charset=utf8", $user, $pass);
            echo "<p class='success'>✓ Connected to MariaDB successfully!</p>";
            
            // Show tables
            $stmt = $pdo->query("SHOW TABLES");
            $tables = $stmt->fetchAll(PDO::FETCH_COLUMN);
            
            if (count($tables) > 0) {
                echo "<h3>Tables in database:</h3>";
                echo "<table><tr><th>Table Name</th></tr>";
                foreach ($tables as $table) {
                    echo "<tr><td>$table</td></tr>";
                }
                echo "</table>";
                
                // Show data from test table
                $stmt = $pdo->query("SELECT * FROM test ORDER BY created_at DESC LIMIT 5");
                $data = $stmt->fetchAll();
                
                if (count($data) > 0) {
                    echo "<h3>Recent test entries:</h3>";
                    echo "<table><tr><th>ID</th><th>Message</th><th>Created</th></tr>";
                    foreach ($data as $row) {
                        echo "<tr><td>{$row['id']}</td><td>{$row['message']}</td><td>{$row['created_at']}</td></tr>";
                    }
                    echo "</table>";
                }
            }
        } catch (PDOException $e) {
            echo "<p class='error'>✗ Connection failed: " . $e->getMessage() . "</p>";
        }
        ?>
        
        <h2>Images Gallery</h2>
        <div>
            <?php
            $image_dir = 'images';
            if (is_dir($image_dir)) {
                $images = glob($image_dir . "/*.{jpg,jpeg,png,gif}", GLOB_BRACE);
                if (count($images) > 0) {
                    foreach ($images as $image) {
                        echo "<img src='$image' alt='Gallery image'>";
                    }
                } else {
                    echo "<p>No images found. Creating sample images...</p>";
                    mkdir($image_dir, 0755, true);
                }
            } else {
                echo "<p>Creating images directory...</p>";
                mkdir($image_dir, 0755, true);
            }
            ?>
        </div>
    </div>
</body>
</html>
EOF
```
Создаем директорию Image и файлы в ней:
```
mkdir -p ~/web/images
echo "Sample image 1" > ~/web/images/image1.txt
echo "Sample image 2" > ~/web/images/image2.txt
```
Устанавливаем необходимое ПО:
```
apt-get update
apt-get install apache2 -y
apt-get install mariadb-server mariadb-client -y
apt-get install php8.2 php8.2-mysqli php8.2-mysqlnd apache2-mod_php8.2 -y
apt-get install php8.2-pdo php8.2-pdo_mysql -y
```
Запускаем MariaDB:
```
systemctl enable --now mariadb
```
Настраиваем безопасность:
```
mysql_secure_installation
```
Ответы:
```
Enter current password for root: [Enter] (просто нажать Enter)
Switch to unix_socket authentication: n
Change the root password? n (или y если хотите установить пароль)
Remove anonymous users? y
Disallow root login remotely? y
Remove test database and access to it? y
Reload privilege tables now? y
```
Заходим в MySql:
```
mysql -u root -p
```
Настраиваем БД:
```
CREATE DATABASE IF NOT EXISTS webdb;
USE webdb;
SOURCE /root/web/dump.sql;
CREATE USER 'web'@'localhost' IDENTIFIED BY 'P@ssw0rd';
GRANT ALL PRIVILEGES ON webdb.* TO 'web'@'localhost';
FLUSH PRIVILEGES;
SHOW DATABASES;
SELECT User, Host FROM mysql.user WHERE User='web';
SHOW TABLES;
SELECT * FROM test;
EXIT;
```
Вывод:

<img width="570" height="662" alt="image" src="https://github.com/user-attachments/assets/0484765a-03b8-400b-8bb9-cb5a5a29ade6" />

Проверяем, что пользователь web может подключиться:
```
mysql -u web -p'P@ssw0rd' -e "SHOW DATABASES;"
mysql -u web -p'P@ssw0rd' -D webdb -e "SHOW TABLES;"
```

<img width="612" height="262" alt="image" src="https://github.com/user-attachments/assets/a40681bd-88e6-4f84-9571-c1d49af64204" />

Настраиваем апач, Проверяем текущий DocumentRoot:
```
httpd2 -S | grep -i "documentroot"
```

<img width="1034" height="54" alt="image" src="https://github.com/user-attachments/assets/751ecc0f-6ac9-408c-8729-84c217476130" />

Создаем директорию htdocs (если её нет) и копируем туда файлы для приложения:
```
mkdir -p /etc/httpd2/htdocs
cp /root/web/index.php /etc/httpd2/htdocs/
cp -r /root/web/images /etc/httpd2/htdocs/
echo "<?php phpinfo(); ?>" > /etc/httpd2/htdocs/info.php
echo "<?php echo 'OK'; ?>" > /etc/httpd2/htdocs/simple.php
```
Устанавливаем владельца и права:
```
chown -R apache:apache /etc/httpd2/htdocs/
find /etc/httpd2/htdocs/ -type d -exec chmod 755 {} \;
find /etc/httpd2/htdocs/ -type f -exec chmod 644 {} \;
```
Проверяем наличие модуля PHP в Apache:
```
ls -la /etc/httpd2/conf/mods-available/ | grep php
```

<img width="682" height="67" alt="image" src="https://github.com/user-attachments/assets/d48694e1-01e6-4fed-971d-61571d91e86f" />

Включаем модуль (создаем симлинки):
```
ln -sf /etc/httpd2/conf/mods-available/mod_php8.2.load /etc/httpd2/conf/mods-enabled/
ln -sf /etc/httpd2/conf/mods-available/mod_php8.2.conf /etc/httpd2/conf/mods-enabled/
```
Проверяем активные конфиги:
```
ls -la /etc/httpd2/conf/sites-enabled/
```

<img width="910" height="127" alt="image" src="https://github.com/user-attachments/assets/c2d4c6b8-5453-48cb-a4ed-10e4d514a8a9" />

Редактируем основной конфиг:
```
vim /etc/httpd2/conf/sites-available/default.conf
```
```
DocumentRoot /etc/httpd2/htdocs
<Directory /etc/httpd2/htdocs>
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>
```

<img width="907" height="578" alt="image" src="https://github.com/user-attachments/assets/8dfbd5b4-c3a4-4467-bf47-87fb37eb9ff3" />

Проверяем конфигурацию:
```
httpd2 -t
```
Если Syntax OK, перезапускаем:
```
systemctl restart httpd2
```
Проверяем статус:
```
systemctl status httpd2 --no-pager | head -10
```
### Все дальнейшие проверки можно через браузер делать
Проверяем простой PHP файл (Должны увидеть: OK):
```
curl http://localhost/simple.php
```
Проверяем информацию о PHP:
```
curl http://localhost/info.php | head -20
```
Создадим тестовый файл для проверки БД:
```
cat > /etc/httpd2/htdocs/db-test.php << 'EOF'
<?php
$host = 'localhost';
$dbname = 'webdb';
$user = 'web';
$pass = 'P@ssw0rd';

echo "<h1>Database Connection Test</h1>";

try {
    $pdo = new PDO("mysql:host=$host;dbname=$dbname", $user, $pass);
    echo "<p style='color:green'>✓ Connected to database successfully!</p>";
    
    $stmt = $pdo->query("SELECT VERSION()");
    $version = $stmt->fetch();
    echo "<p>MySQL Version: " . $version[0] . "</p>";
    
    $stmt = $pdo->query("SHOW TABLES");
    echo "<h3>Tables:</h3><ul>";
    while ($row = $stmt->fetch()) {
        echo "<li>" . $row[0] . "</li>";
    }
    echo "</ul>";
    
} catch (PDOException $e) {
    echo "<p style='color:red'>✗ Connection failed: " . $e->getMessage() . "</p>";
}
?>
EOF

chown apache:apache /etc/httpd2/htdocs/db-test.php
```
Проверим его:
```
curl http://localhost/db-test.php
```
Настраиваем Iptables (под вопросом насколько нужно):
```
iptables -I INPUT -p tcp --dport 80 -j ACCEPT
mkdir /etc/iptables
iptables-save > /etc/iptables/rules.v4
```

### По итогу должно получиться следующее (на других узлах тоже должно работать):

<img width="1716" height="920" alt="image" src="https://github.com/user-attachments/assets/421813e1-f0a5-48cb-af14-ac494d56d56b" />

<img width="1718" height="916" alt="image" src="https://github.com/user-attachments/assets/6be2eb0b-f86e-4b69-98ea-c04f2885035f" />

<img width="1716" height="914" alt="image" src="https://github.com/user-attachments/assets/b3d37a35-86f7-4527-aca4-33c7a8b7feea" />

<img width="1718" height="915" alt="image" src="https://github.com/user-attachments/assets/53d0f0dd-c17f-46b0-a9cc-4db720fe7384" />

<img width="1718" height="960" alt="image" src="https://github.com/user-attachments/assets/f60a68a0-f17d-4dec-bc94-1797fd39652a" />

</details>
 
##  На маршрутизаторах сконфигурируйте статическую трансляцию портов

<details>
    <summary>ЗАДАНИЕ</summary>
 
• Пробросьте порт 8080в порт приложения testapp BR-SRV на маршрутизаторе BR-RTR, для обеспечения работы приложения testapp извне

• Пробросьте порт 8080в порт веб приложения на HQ-SRV на маршрутизаторе HQ-RTR, для обеспечения работы веб приложения извне 

• Пробросьте порт 2026на маршрутизаторе HQ-RTR в порт 2026сервера HQ-SRV, для подключения к серверу по протоколу ssh из внешних сетей 

• Пробросьте порт 2026на маршрутизаторе BR-RTR в порт 2026сервера BR-SRV, для подключения к серверу по протоколу ssh из внешних сетей.

 </details>

<details>
    <summary>НАЖМИ</summary>

На BR-RTR:
```
iptables -t nat -A PREROUTING -i ens33-p tcp --dport 8080 -j DNAT --to-destination 192.168.2.14:8080
iptables -t nat -A PREROUTING -i ens33 -p tcp --dport 2026 -j DNAT --to-destination 192.168.2.14:2026
sysctl -p iptables-save > /etc/iptables/rules.v4
```
На HQ-RTR:
```
iptables -t nat -A PREROUTING -i ens33-p tcp --dport 8080 -j DNAT --to-destination 192.168.2.62:8080
iptables -t nat -A PREROUTING -i ens33 -p tcp --dport 2026 -j DNAT --to-destination 192.168.1.62:2026
sysctl -p
 iptables-save > /etc/iptables/rules.v4
```
Проверяем ssh на HQ-CLI (должны подключиться к BR-SRV):
```
ssh -p 2026 sshuser@172.16.2.2
```


 </details>

 ## Настройте веб-сервер nginx как обратный прокси-сервер на ISP
 
<details>
    <summary>ЗАДАНИЕ</summary>


При обращении по доменному имени web.au-team.irpo у клиента должно открываться веб приложение на HQ-SRV

• При обращении по доменному имени docker.au-team.irpo клиента должно открываться веб приложение testapp

 </details>

<details>
    <summary>НАЖМИ</summary>

### Чтобы ISP видел HQ-SRV нужно прокинуть на него frr, а если прокинуть на него маршрутизацию, то вся сеть начинает невероятно плохо работать, так что на свой страх и риск

Устанавливаем nginx:
```
apt-get install nginx -y
```
Запускаем:
```
systemctl enable --now nginx
```
Редачим /etc/nginx/nginx.conf (Вставляем перед последней скобкой, иначе nginx будет жаловаться на синтаксис):
```
server
{
listen 80;
server_name web.au.team;
location / {
proxy_pass http://192.168.1.62:80;
}
}
server {
listen 80;
server_name docker;
location / {
proxy_pass http://192.168.2.14:8080;
}
}
```

<img width="991" height="701" alt="image" src="https://github.com/user-attachments/assets/e6e43ee4-1744-4e2a-851b-0df2811207bb" />

Проверяем конфиг и перезапускаем:
```
nginx -t
systemctl restart nginx
```
### Должно получиться так (Может долго очень грузится и не на всех хостах и причина этому скорее всего - ISP, маршрутизация начинает очень плохо работать с 3 включенным роутером):

<img width="1721" height="949" alt="image" src="https://github.com/user-attachments/assets/8eb0e1c3-0b27-4ad5-8247-2e5bd164175b" />

<img width="1718" height="951" alt="image" src="https://github.com/user-attachments/assets/3e9ae7fd-29c0-43d5-9350-53b20178918a" />

 </details>

## На маршрутизаторе ISP настройте web-based аутентификацию

<details>
    <summary>ЗАДАНИЕ</summary>

При обращении к сайту web.au-team.irpo клиенту должно быть предложено ввести аутентификационные данные 

• В качестве логина для аутентификации выберите WEBс паролем P@ssw0rd 

• Выберите файл /etc/nginx/.htpasswd в качестве хранилища учётных записей 

• При успешной аутентификации клиент должен перейти на веб сайт.

 </details>

<details>
    <summary>НАЖМИ</summary>

Создаем файл .htpasswd с пользователем WEB:
```
htpasswd -bc /etc/nginx/.htpasswd WEB P@ssw0rd
```
Проверяем содержимое файла:
```
cat /etc/nginx/.htpasswd
```
Устанавливаем владельца и права:
```
chown root:root /etc/nginx/.htpasswd
chmod 644 /etc/nginx/.htpasswd
```

Редактируем конфигурацию /etc/nginx/sites-available.d/web.au.team:
```
server {
    listen 80;
    server_name web.au.team;
    access_log /var/log/nginx/web.au.team.access.log;
    error_log /var/log/nginx/web.au.team.error.log;
    location / {
        auth_basic "Restricted Access - Please Login";
        auth_basic_user_file /etc/nginx/.htpasswd;
        proxy_pass http://192.168.1.62;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Authorization $http_authorization;
        proxy_pass_header Authorization;
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
}
```
Создаем симлинк в sites-enabled.d:
```
ln -s /etc/nginx/sites-available.d/web.au.team /etc/nginx/sites-enabled.d/
```
Проверяем синтаксис и перезапускаем nginx:
```
nginx -t
systemctl restart nginx
```
На клиентской машине, с которой будете тестировать, добавьте:
```
echo "192.168.131.189 web.au.team" >> /etc/hosts
```
На DNS-сервере добавьте A-запись:
```
web.au.team. IN A 192.168.131.189
```
Включаем форвардинг пакетов:
```
echo 1 > /proc/sys/net/ipv4/ip_forward
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p
```
Настраиваем iptables:
```
iptables -t nat -A PREROUTING -p tcp --dport 80 -d 192.168.131.189 -j DNAT --to-destination 192.168.1.62:80
iptables -t nat -A PREROUTING -p tcp --dport 80 -d 172.16.1.1 -j DNAT --to-destination 192.168.1.62:80
iptables -t nat -A PREROUTING -p tcp --dport 80 -d 172.16.2.1 -j DNAT --to-destination 192.168.1.62:80
iptables -A FORWARD -p tcp -d 192.168.1.62 --dport 80 -j ACCEPT
iptables -A FORWARD -p tcp -s 192.168.1.62 --sport 80 -j ACCEPT
iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE
iptables-save > /etc/iptables/rules.v4
```

</details>

# Модуль 3
##  Выполните импорт пользователей в домен au-team.irpo
<details>
    <summary>ЗАДАНИЕ</summary>

В качестве файла источника выберите файл users.csv располагающийся в образе Additional.iso 

• Пользователи должны быть импортированы со своими паролями и другими атрибутами 

• Убедитесь, что импортированные пользователи могут войти на машину HQ-CLI

 </details>
 <details>
     <summary>НАЖМИ</summary>

Проверяем что домен работает и получаем билет если не получали:
```
systemctl status samba
samba-tool domain level show
kinit administrator@AU.TEAM
klist
```
Создаем users.csv (если нет)(ПРОБЕЛОВ БЫТЬ НЕ ДОЛЖНО):
```
cat > /tmp/users.csv << 'EOF'
login,password,givenname,surname,department,mail
ivanov,P@ssw0rd,Ivan,Ivanov,IT,ivan.ivanov@au.team
petrov,P@ssw0rd,Petr,Petrov,Sales,petr.petrov@au.team
sidorov,P@ssw0rd,Sidor,Sidorov,HR,sidor.sidorov@au.team
smirnova,P@ssw0rd,Anna,Smirnova,Finance,anna.smirnova@au.team
kuznetsov,P@ssw0rd,Nikolai,Kuznetsov,IT,nikolai.kuznetsov@au.team
popova,P@ssw0rd,Elena,Popova,Marketing,elena.popova@au.team
volkov,P@ssw0rd,Alexey,Volkov,Sales,alexey.volkov@au.team
sokolov,P@ssw0rd,Dmitry,Sokolov,IT,dmitry.sokolov@au.team
morozova,P@ssw0rd,Olga,Morozova,HR,olga.morozova@au.team
novikov,P@ssw0rd,Michael,Novikov,Finance,michael.novikov@au.team
EOF
```
Создаем скрипт для импорта юзеров /root/import_users.sh:
```
#!/bin/bash

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
CYAN='\033[0;36m'
NC='\033[0m'

# Configuration
INPUT_FILE="/tmp/users.csv"
DOMAIN="au.team"
SEPARATOR=","
LOG_FILE="/var/log/import_users_$(date +%Y%m%d_%H%M%S).log"
PASSWORD="P@ssw0rd"

log() {
    echo -e "$1" | tee -a "$LOG_FILE"
}

clear

log "${BLUE}========================================${NC}"
log "${BLUE}    IMPORT USERS TO DOMAIN au.team     ${NC}"
log "${BLUE}========================================${NC}"
log "${GREEN}Password for all users: ${YELLOW}${PASSWORD}${NC}"
log "${GREEN}Mode: ${YELLOW}NO password change required${NC}"
log "${BLUE}----------------------------------------${NC}"

if [ "$EUID" -ne 0 ]; then
    log "${RED}Error: Run as root${NC}"
    exit 1
fi

if ! command -v samba-tool &> /dev/null; then
    log "${RED}Error: samba-tool not found${NC}"
    exit 1
fi

if [ ! -f "$INPUT_FILE" ]; then
    log "${RED}Error: File $INPUT_FILE not found${NC}"
    exit 1
fi

HEADER=$(head -1 "$INPUT_FILE")
if [[ "$HEADER" != *"login"* ]]; then
    log "${YELLOW}Warning: Invalid file structure${NC}"
    log "Header: $HEADER"
    log "Expected: login,givenname,surname,department,mail"
    read -p "Continue? (y/n): " -n 1 -r
    echo
    if [[ ! $REPLY =~ ^[Yy]$ ]]; then
        exit 1
    fi
fi

log "${GREEN}All checks passed${NC}"
log "Input file: ${CYAN}$INPUT_FILE${NC}"
log "Log file: ${CYAN}$LOG_FILE${NC}"
log "${BLUE}----------------------------------------${NC}"

TOTAL=0
SUCCESS=0
FAILED=0
SKIPPED=0
START_TIME=$(date +%s)

EXISTING_USERS=$(samba-tool user list)

tail -n +2 "$INPUT_FILE" | while IFS="$SEPARATOR" read -r login givenname surname department mail
do
    TOTAL=$((TOTAL + 1))
    
    login=$(echo "$login" | tr -d '\r' | sed 's/^[[:space:]]*//;s/[[:space:]]*$//')
    givenname=$(echo "$givenname" | tr -d '\r' | sed 's/^[[:space:]]*//;s/[[:space:]]*$//')
    surname=$(echo "$surname" | tr -d '\r' | sed 's/^[[:space:]]*//;s/[[:space:]]*$//')
    department=$(echo "$department" | tr -d '\r' | sed 's/^[[:space:]]*//;s/[[:space:]]*$//')
    mail=$(echo "$mail" | tr -d '\r' | sed 's/^[[:space:]]*//;s/[[:space:]]*$//')
    
    if [ -z "$login" ]; then
        log "${YELLOW}Line $TOTAL: skipped (empty login)${NC}"
        SKIPPED=$((SKIPPED + 1))
        continue
    fi
    
    if [ -z "$givenname" ]; then
        givenname="$login"
    fi
    
    if [ -z "$surname" ]; then
        surname="-"
    fi
    
    if [ -z "$mail" ]; then
        mail="${login}@${DOMAIN}"
    fi
    
    if [ -z "$department" ]; then
        department="Users"
    fi
    
    log "${BLUE}[$TOTAL]${NC} Processing: ${CYAN}$login${NC} ($givenname $surname) [$department]"
    
    if echo "$EXISTING_USERS" | grep -q "^$login$"; then
        log "${YELLOW}  User $login already exists, skipping${NC}"
        SKIPPED=$((SKIPPED + 1))
        continue
    fi
    
    samba-tool user create "$login" "$PASSWORD" \
        --given-name="$givenname" \
        --surname="$surname" \
        --mail-address="$mail" \
        --department="$department" \
        --login-shell="/bin/bash" \
        --home-drive="H:" \
        --home-directory="\\\\${DOMAIN}\\users\\$login" >> "$LOG_FILE" 2>&1
    
    if [ $? -eq 0 ]; then
        log "${GREEN}  Created: $login (password: $PASSWORD)${NC}"
        SUCCESS=$((SUCCESS + 1))
        
        samba-tool group addmembers "Domain Users" "$login" &>/dev/null
        samba-tool user setpassword "$login" --newpassword="$PASSWORD" >> "$LOG_FILE" 2>&1
        
        if [ "$department" != "Users" ] && [ -n "$department" ]; then
            if samba-tool group list | grep -q "^$department$"; then
                samba-tool group addmembers "$department" "$login" &>/dev/null
                log "    Added to group: $department"
            fi
        fi
    else
        log "${RED}  Failed to create $login${NC}"
        FAILED=$((FAILED + 1))
    fi
    
    sleep 0.5
    
done

END_TIME=$(date +%s)
DURATION=$((END_TIME - START_TIME))

log "${BLUE}========================================${NC}"
log "${GREEN}IMPORT COMPLETED${NC}"
log "${BLUE}----------------------------------------${NC}"
log "Duration: ${CYAN}${DURATION} sec${NC}"
log "Total processed: ${CYAN}$TOTAL${NC}"
log "Successfully created: ${GREEN}$SUCCESS${NC}"
log "Skipped: ${YELLOW}$SKIPPED${NC}"
log "Failed: ${RED}$FAILED${NC}"
log "${BLUE}----------------------------------------${NC}"
log "Log saved: ${CYAN}$LOG_FILE${NC}"
log "${BLUE}========================================${NC}"

if [ $SUCCESS -gt 0 ]; then
    echo ""
    log "${GREEN}Last created users:${NC}"
    samba-tool user list | tail -$(( SUCCESS > 5 ? 5 : SUCCESS )) | while read user; do
        echo "   $user"
    done
fi

if [ $SUCCESS -gt 0 ]; then
    echo ""
    log "${GREEN}Password status:${NC}"
    samba-tool user list | tail -$(( SUCCESS > 3 ? 3 : SUCCESS )) | while read user; do
        if samba-tool user show "$user" | grep -q "user must change password: FALSE"; then
            log "   $user: OK"
        else
            log "   $user: WARNING"
        fi
    done
fi
```
Выполняем:
```
chmod +x /root/import_users.sh
/root/import_users.sh
```
### Если нужно удалить юзеров можем сделать скрипт на удаление /root/delete_users.sh:
```
#!/bin/bash

INPUT_FILE="/tmp/users.csv"
SEPARATOR=","
LOG_FILE="/var/log/delete_users_$(date +%Y%m%d_%H%M%S).log"

if [ "$EUID" -ne 0 ]; then
    echo "Run as root"
    exit 1
fi

if [ ! -f "$INPUT_FILE" ]; then
    echo "File $INPUT_FILE not found"
    exit 1
fi

echo "WARNING: This will delete users from domain!"
echo "Press Ctrl+C to cancel or ENTER to continue"
read

TOTAL=0
DELETED=0
FAILED=0
NOT_FOUND=0

EXISTING_USERS=$(samba-tool user list)

tail -n +2 "$INPUT_FILE" | while IFS="$SEPARATOR" read -r login rest
do
    TOTAL=$((TOTAL + 1))
    login=$(echo "$login" | tr -d '\r' | xargs)
    
    if [ -z "$login" ]; then
        continue
    fi
    
    echo "[$TOTAL] Processing: $login"
    
    if echo "$EXISTING_USERS" | grep -q "^$login$"; then
        samba-tool user delete "$login" >> "$LOG_FILE" 2>&1
        if [ $? -eq 0 ]; then
            echo "  -> Deleted: $login"
            DELETED=$((DELETED + 1))
        else
            echo "  -> Error deleting: $login"
            FAILED=$((FAILED + 1))
        fi
    else
        echo "  -> Not found: $login"
        NOT_FOUND=$((NOT_FOUND + 1))
    fi
    
    sleep 0.5
done

echo "==================================="
echo "DELETE COMPLETED"
echo "Total processed: $TOTAL"
echo "Deleted: $DELETED"
echo "Not found: $NOT_FOUND"
echo "Failed: $FAILED"
echo "Log: $LOG_FILE"
```
Выполняем:
```
chmod +x /root/delete_users.sh
/root/delete_users.sh
```

</details>

## Выполните настройку центра сертификации на базе HQ-SRV

<details>
    <summary>ЗАДАНИЕ</summary>

Необходимо использовать отечественные алгоритмы шифрования 

• Сертификаты выдаются на 30дней 

• Обеспечьте доверие сертификату для HQ-CLI 

• Выдайте сертификаты веб серверам 

• Перенастройте ранее настроенный реверсивный прокси nginx на протокол https 

• При обращении к веб серверам https://web.au-team.irpo и https://docker.au-team.irpo у браузера клиента не должно возникать предупреждений.

 </details>

 <details>
    <summary>НАЖМИ</summary>
  
Создаем структуру CA в домашней директории:
```
cd ~
mkdir -p ~/ca/{certs,crl,newcerts,private}
chmod 700 ~/ca/private
touch ~/ca/index.txt
echo 1000 > ~/ca/serial
```
Редачим конфиг /var/lib/ssl/openssl.cnf:
```

<img width="663" height="892" alt="image" src="https://github.com/user-attachments/assets/1d5eab13-0e47-4855-8481-efd711c2d501" />

<img width="704" height="835" alt="image" src="https://github.com/user-attachments/assets/660ab021-aa65-46e3-b3c7-f3842010b084" />
```
Проверяем конфиг:
```
openssl version -d
openssl ciphers -v
```
Создаем и проверяем корневой сертификат СА (вводить построчно):
```
cd ~/ca
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 \
  -out private/ca.key.pem
chmod 400 private/ca.key.pem

openssl req -x509 -new -key private/ca.key.pem \
  -days 30 -sha256 \
  -subj "/C=RU/ST=Moscow/L=Moscow/O=MyCompany/CN=HQ-CA/emailAddress=admin@au.team" \
  -out certs/ca.cert.pem
  
openssl x509 -in certs/ca.cert.pem -text -noout | grep -E "Issuer:|Subject:|Not Before|Not After"
```
Создаем сертификаты для HQ-SRV и BR-SRV (вводить построчно):
```
cd ~/ca

openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 \
  -out private/web.au.team.key.pem
chmod 400 private/web.au.team.key.pem

openssl req -new -key private/web.au.team.key.pem \
  -subj "/C=RU/ST=Moscow/L=Moscow/O=MyCompany/CN=web.au.team/emailAddress=admin@au.team" \
  -out web.au.team.csr

openssl x509 -req -in web.au.team.csr \
  -CA certs/ca.cert.pem -CAkey private/ca.key.pem \
  -CAcreateserial -out certs/web.au.team.cert.pem \
  -days 30 -sha256

openssl verify -CAfile certs/ca.cert.pem certs/web.au.team.cert.pem

openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 \
  -out private/docker.au.team.key.pem
chmod 400 private/docker.au.team.key.pem

openssl req -new -key private/docker.au.team.key.pem \
  -subj "/C=RU/ST=Moscow/L=Moscow/O=MyCompany/CN=docker.au.team/emailAddress=admin@au.team" \
  -out docker.au.team.csr

openssl x509 -req -in docker.au.team.csr \
  -CA certs/ca.cert.pem -CAkey private/ca.key.pem \
  -CAcreateserial -out certs/docker.au.team.cert.pem \
  -days 30 -sha256

openssl verify -CAfile certs/ca.cert.pem certs/docker.au.team.cert.pem
```
Устанавливаем сертификат на HQ-SRV:
```
mkdir -p /etc/apache2/ssl
cp ~/ca/certs/web.au.team.cert.pem /etc/apache2/ssl/
cp ~/ca/private/web.au.team.key.pem /etc/apache2/ssl/
cp ~/ca/certs/ca.cert.pem /etc/apache2/ssl/
a2enmod ssl
```
Подгатавливаем и отправляем архивы для BR-SRV и HQ-RTR:
```
cd ~/ca
tar -czf docker_certs.tar.gz certs/docker.au.team.cert.pem private/docker.au.team.key.pem certs/ca.cert.pem
scp -P 2026 docker_certs.tar.gz sshuser@192.168.2.14:/tmp/
tar -czf hq-rtr_certs.tar.gz certs/ca.cert.pem
scp -P 2026 hq-rtr_certs.tar.gz sshuser@192.168.1.1:/tmp/
cp /root/ca/certs/web.au.team.cert.pem /home/sshuser/
cp /root/ca/private/web.au.team.key.pem /home/sshuser/
cp /root/ca/certs/docker.au.team.cert.pem /home/sshuser/
cp /root/ca/private/docker.au.team.key.pem /home/sshuser/
chown sshuser:sshuser /home/sshuser/*.pem
```
На BR-SRV распаковываем:
```
cd /tmp
tar -xzf docker_certs.tar.gz
mkdir -p /etc/nginx/ssl
cp certs/docker.au.team.cert.pem /etc/nginx/ssl/
cp certs/ca.cert.pem /etc/nginx/ssl/
cp private/docker.au.team.key.pem /etc/nginx/ssl/
chmod 644 /etc/nginx/ssl/*.pem
chmod 600 /etc/nginx/ssl/docker.au.team.key.pem
ls -la /etc/nginx/ssl/
```
На HQ-RTR распаковываем:
```
cd /tmp
tar -xzf hq-rtr_certs.tar.gz
mkdir -p /etc/nginx/ssl
cp certs/ca.cert.pem /etc/nginx/ssl/
scp -P 2026 sshuser@192.168.1.62:/home/sshuser/web.au.team.cert.pem /tmp/
scp -P 2026 sshuser@192.168.1.62:/home/sshuser/web.au.team.key.pem /tmp/
scp -P 2026 sshuser@192.168.1.62:/home/sshuser/docker.au.team.cert.pem /tmp/
scp -P 2026 sshuser@192.168.1.62:/home/sshuser/docker.au.team.key.pem /tmp/
cp /tmp/web.au.team.cert.pem /etc/nginx/ssl/
cp /tmp/web.au.team.key.pem /etc/nginx/ssl/
cp /tmp/docker.au.team.cert.pem /etc/nginx/ssl/
cp /tmp/web.docker.team.key.pem /etc/nginx/ssl/
```
На HQ-RTR редактируем nginx /etc/nginx/nginx.conf:
```
server {
    listen 80;
    server_name web.au.team;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name web.au.team;
    
    ssl_certificate /etc/nginx/ssl/web.au.team.cert.pem;
    ssl_certificate_key /etc/nginx/ssl/web.au.team.key.pem;
    ssl_trusted_certificate /etc/nginx/ssl/ca.cert.pem;
    
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    
    location / {
        proxy_pass http://192.168.1.62:80;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
server {
    listen 80;
    server_name docker.au.team;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name docker.au.team;
    
    ssl_certificate /etc/nginx/ssl/docker.au.team.cert.pem;
    ssl_certificate_key /etc/nginx/ssl/docker.au.team.key.pem;
    ssl_trusted_certificate /etc/nginx/ssl/ca.cert.pem;
    
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    
    location / {
        proxy_pass http://192.168.2.14:80;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
Устанавливаем доверие на HQ-CLI:
```
cp ~/ca/certs/ca.cert.pem ~/ca/certs/CA-HQ.crt
scp -P 2026 ~/ca/certs/CA-HQ.crt sshuser@192.168.1.3:/tmp/
```
Получаем сертификат на HQ-CLI:
```
cp /tmp/CA-HQ.crt /etc/pki/ca-trust/source/anchors/
update-ca-trust
```
Проверяем:
```
curl -I https://web.au.team
curl -I https://docker.au.team
openssl s_client -connect web.au.team:443 -showcerts < /dev/null 2>/dev/null | openssl x509 -text | grep -E "Issuer:|Subject:|Not Before|Not After"
```

 </details>

## Перенастройте ip-туннель с базового до уровня туннеля, обеспечивающего шифрование трафика 

<details>
    <summary>ЗАДАНИЕ</summary>

Настройте защищенный туннель между HQ-RTR и BR-RTR 

• Внесите необходимые изменения в конфигурацию динамической маршрутизации, протокол динамической маршрутизации должен возобновить работу после перенастройки туннеля 

• Выбранное программное обеспечение, обоснование его выбора и его основные параметры, изменения в конфигурации динамической маршрутизации отметьте в отчёте.

 </details>

 <details>
    <summary>НАЖМИ</summary>

Устанавливаем Strongswan:
```
apt-get install strongswan
```
Редачим конфиг /etc/strongswan/ipsec.conf:
### HQ-RTR
```
config setup
    charondebug="ike 2, knl 2, cfg 2"

conn gre-tunnel
    left=172.16.1.2
    leftid=172.16.1.2
    right=172.16.2.2
    rightid=172.16.2.2
    type=transport
    keyexchange=ikev2
    authby=secret
    esp=aes256-sha256-modp2048
    ikelifetime=24h
    lifetime=8h
    dpddelay=10s
    dpdtimeout=30s
    dpdaction=restart
    auto=start
```
### BR-RTR:
```
config setup
    charondebug="ike 2, knl 2, cfg 2"

conn gre-tunnel
    left=172.16.2.2
    leftid=172.16.2.2
    right=172.16.1.2
    rightid=172.16.1.2
    type=transport
    keyexchange=ikev2
    authby=secret
    esp=aes256-sha256-modp2048
    ikelifetime=24h
    lifetime=8h
    dpddelay=10s
    dpdtimeout=30s
    dpdaction=restart
    auto=start
```
Создаем файл с ключом на обоих роутерах:
```
: PSK "very_strong_secret_key_change_this_123456"
```
Добавляем строчку в файл /etc/strongswan/ipsec.secrets:
```
172.16.1.2 172.16.2.2 : PSK "TestPassword123"
```
Изменяем права к файлу:
```
chmod 600 /etc/strongswan/ipsec.secrets
chown root:root /etc/strongswan/ipsec.secrets
```
Настраиваем Iptables на обоих роутерах:
```
iptables -I INPUT -i ens33 -p gre -j ACCEPT
iptables -I OUTPUT -o ens33 -p gre -j ACCEPT
iptables -I INPUT -i ens33 -p udp --dport 500 -j ACCEPT
iptables -I INPUT -i ens33 -p udp --dport 4500 -j ACCEPT
iptables -I INPUT -i ens33 -p esp -j ACCEPT
iptables -I INPUT -i ens33 -p ah -j ACCEPT
iptables-save > /etc/iptables/rules.v4
```
Настраиваем OSPF:
### HQ-RTR:
```
vtysh
configure terminal
interface tun1
 ip ospf network point-to-point
 ip ospf hello-interval 10
 ip ospf dead-interval 40
router ospf
 network 10.10.10.0/30 area 0
 network 192.168.1.0/26 area 0
do wr
```
### BR-RTR:
```
vtysh
configure terminal
interface tun1
 ip ospf network point-to-point
 ip ospf hello-interval 10
 ip ospf dead-interval 40
router ospf
 network 10.10.10.0/30 area 0
 network 192.168.2.0/28 area 0
do wr
```
Перезапускаем сервисы и проверяем:
```
systemctl restart frr
systemctl enable strongswan-starter
systemctl start strongswan-starter
systemctl status strongswan-starter
ipsec status
```

<img width="816" height="88" alt="image" src="https://github.com/user-attachments/assets/6b437d94-946f-444d-92e0-7c7210086f83" />

 </details>

## Настройте межсетевой экран на маршрутизаторах HQ-RTR и BR-RTR на сеть в сторону ISP 

<details>
    <summary>ЗАДАНИЕ</summary>

Обеспечьте работу протоколов http, https, dns, ntp, icmp или дополнительных нужных протоколов 

• Запретите остальные подключения из сети Интернет во внутреннюю сеть.

 </details>

<details>
    <summary>НАЖМИ</summary>

Настраиваем Iptables на HQ-RTR:
```
iptables -F
iptables -X
iptables -t nat -F
iptables -t mangle -F
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT
iptables -I INPUT 1 -m conntrack --ctstate INVALID -j DROP
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -i ens37 -p icmp --icmp-type echo-request -j ACCEPT
iptables -A INPUT -i ens33 -p icmp --icmp-type echo-request -j ACCEPT
iptables -A INPUT -i ens37 -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -i ens37 -p tcp --dport 2026 -j ACCEPT
iptables -A INPUT -i tun1 -p 89 -j ACCEPT
iptables -A INPUT -i tun1 -p icmp --icmp-type echo-request -j ACCEPT
iptables -I FORWARD 1 -m conntrack --ctstate INVALID -j DROP
iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -A FORWARD -i ens37 -o tun1 -j ACCEPT
iptables -A FORWARD -i tun1 -o ens37 -j ACCEPT
iptables -A FORWARD -i ens37 -o ens33 -j ACCEPT
iptables -A FORWARD -i tun1 -o ens37 -p tcp --dport 80 -d 192.168.1.62 -j ACCEPT
iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE
iptables-save > /etc/iptables/rules.v4
```
Настраиваем Iptables на BR-RTR:
```
iptables -F
iptables -X
iptables -t nat -F
iptables -t mangle -F
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT
iptables -I INPUT 1 -m conntrack --ctstate INVALID -j DROP
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -i ens37 -p icmp --icmp-type echo-request -j ACCEPT
iptables -A INPUT -i ens33 -p icmp --icmp-type echo-request -j ACCEPT
iptables -A INPUT -i ens37 -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -i ens37 -p tcp --dport 2026 -j ACCEPT
iptables -A INPUT -i tun1 -p 89 -j ACCEPT
iptables -A INPUT -i tun1 -p icmp --icmp-type echo-request -j ACCEPT
iptables -I FORWARD 1 -m conntrack --ctstate INVALID -j DROP
iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -A FORWARD -i ens37 -o tun1 -j ACCEPT
iptables -A FORWARD -i tun1 -o ens37 -j ACCEPT
iptables -A FORWARD -i ens37 -o ens33 -j ACCEPT
iptables -A FORWARD -i tun1 -o ens37 -p tcp --dport 8080 -d 192.168.2.14 -j ACCEPT
iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE
iptables-save > /etc/iptables/rules.v4
```
 </details>

## Настройте принт-сервер cups на сервере HQ-SRV

<details>
    <summary>ЗАДАНИЕ</summary>

Опубликуйте виртуальный pdf-принтер 

• На клиенте HQ-CLI подключите виртуальный принтер как принтер по умолчанию.

 </details>

<details>
    <summary>НАЖМИ</summary>

Устанавливаем cups:
```
apt-get update
apt-get install cups cups-pdf
```
Создаем группу и добавляем туда юзера (после этого необходимо перелогиниться):
```
groupadd -r lpadmin
usermod -aG lpadmin $USER
```
Проверяем что драйвер доступен:
```
lpinfo --make-and-model "PDF" -m
```
Настраиваем доступ в консоль и запускаем cups:
```
cupsctl --remote-admin
sudo systemctl restart cups
sudo systemctl enable cups
sudo systemctl status cups
```
Открываем в браузере (login - root, passwd - toor):
```
http://localhost:631/admin
```
Жмем:
```
Добавить принтер
CUPS-PDF (Virtual PDF Printer)
Название Virtual_PDF
Расположение hq-srv
Разрешить совместный доступ
Создать: Generic
Модель: Generic CUPS-PDF Printer (w/ options) (en)
```
Назначаем принтер по умолчанию:
```
lpadmin -d Virtual_PDF
lpstat -d
```
Редактируем /etc/cups/cups-pdf.conf:
```
Меняем Out ${DESKTOP} на:
Out /root/PDF
```
Создаем директорию и перезапускаем cups:
```
mkdir -p /root/PDF
chmod 755 /root/PDF
systemctl restart cups
```
Проводим тестовую печать:
```
echo "Test print job" > ~/test.txt
lp -d Virtual_PDF ~/test.txt
ls -la root/PDF/
```
### Должен появится файл test_job.pdf

На HQ-CLI устанавливаем cups:
```
apt-get update
apt-get install cups cups-filters cups-pk-helper
```
Создаем директорию для печати:
```
mkdir -p ~/.cups
chmod 755 ~/.cups
echo "ServerName 192.168.1.62" > ~/.cups/client.conf
chmod 644 ~/.cups/client.conf
ls -la ~/.cups/
cat ~/.cups/client.conf
```
Проверяем доступность:
```
curl -I http://192.168.1.100:631
lpstat -s
```
Устанавливаем как принтер по умолчанию:
```
lpoptions -d Virtual_PDF
lpstat -d
```
Проводим тестовую печать:
```
echo "Remote print test from HQ-CLI" > ~/remote-test.txt
lp -d Virtual_PDF ~/remote-test.txt
lpstat -o
```
Проверяем что файл появился:
```
ls /root/PDF
```
### Если не сработало можно попробовать сделать следующее:

HQ-SRV:
```
apt-get update
apt-get install cups cups-pdf
usermod -aG lpadmin $USER
sed -i 's|^Out .*|Out /var/spool/cups-pdf/${USER}|' /etc/cups/cups-pdf.conf
cat > /etc/cups/cupsd.conf << 'EOF'
Port 631
Listen 0.0.0.0:631
<Location />
  Order allow,deny
  Allow localhost
  Allow 192.168.1.0/26
  AuthType None
</Location>
<Location /printers>
  Order allow,deny
  Allow localhost
  Allow 192.168.1.0/26
  AuthType None
</Location>
<Location /printers/Virtual_PDF>
  Order allow,deny
  Allow localhost
  Allow 192.168.1.0/26
  AuthType None
</Location>
EOF
lpadmin -x Virtual_PDF 2>/dev/null
lpadmin -p Virtual_PDF -v cups-pdf:/ -E -o printer-is-shared=true
lpadmin -d Virtual_PDF
systemctl restart cups
systemctl enable cups
systemctl status cups
lpstat -p Virtual_PDF -d
```
На HQ-CLI:
```
apt-get update
apt-get install cups-client
mkdir -p ~/.cups
echo "ServerName 192.168.1.62" > ~/.cups/client.conf
chmod 644 ~/.cups/client.conf
lpstat -h 192.168.1.62 -s
lpoptions -h 192.168.1.62 -d Virtual_PDF
lpstat -d
echo "test HQ-CLI" > ~/test-print.txt
lp -h 192.168.1.62 -d Virtual_PDF ~/test-print.txt
```
Проверяем на сервере:
```
find / -name "*.pdf" -mmin -5 2>/dev/null
ls -la /var/spool/cups-pdf/ANONYMOUS/
file /var/spool/cups-pdf/ANONYMOUS/test-print.txt.pdf
```
### Возможные ошибки
Если клиент не видит принтер:
```
rm -rf ~/.cups ~/.cache/cups
mkdir -p ~/.cups
echo "ServerName 192.168.1.62" > ~/.cups/client.conf
lpstat -h 192.168.1.62 -s
```
Если печать не создает PDF:
```
tail -f /var/log/cups/error_log
ls -la /var/spool/cups-pdf/
ls -la /root/PDF/
```
Если ошибка "Запрещено":
```
grep -i "authtype" /etc/cups/cupsd.conf
iptables -F
```
Результаты проверки:
```
# На сервере
lpstat -p Virtual_PDF -d

# На клиенте
lpstat -h 192.168.1.62 -s

# Созданный PDF-файл
ls -la /var/spool/cups-pdf/ANONYMOUS/
```

</details>

## Реализуйте логирование при помощи rsyslog на устройствах HQ-RTR, BR-RTR, BR-SRV

<details>
    <summary>ЗАДАНИЕ</summary>

 Сервер сбора логов расположен на HQ-SRV, убедитесь, что сервер не является клиентом самому себе 
 
• Приоритет сообщений должен быть не ниже warning 

• Все журналы должны находиться в директории /opt. Для каждого устройства должна выделяться своя поддиректория, которая совпадает с именем машины 

• Реализуйте ротацию собранных логов на сервере HQ-SRV: 

• Ротируются все логи, находящиеся в директории и поддиректориях /opt 

• Ротация производится один раз в неделю 

• Логи необходимо сжимать 

• Минимальный размер логов для ротации – 10МБ.

 </details>

 <details>
    <summary>НАЖМИ</summary>
  
### Настраиваем HQ-RTR:
Устанавливаем rsyslog:
```
apt-get update && apt-get install rsyslog -y
```
Создаем файл для загрузки модулей:
```
cat > /etc/rsyslog.d/00-modules.conf << 'EOF'
module(load="imudp")
module(load="imtcp")
EOF
```
Настраиваем прием логов:
```
cat > /etc/rsyslog.d/01-inputs.conf << 'EOF'
input(type="imudp" port="514" ruleset="remote")
input(type="imtcp" port="514" ruleset="remote")
EOF
```
Настраиваем шаблоны и правила:
```
cat > /etc/rsyslog.d/10-remote-rules.conf << 'EOF'
template(name="RemotePerHostLogs" type="string"
         string="/opt/%HOSTNAME%/%programname%.log")

ruleset(name="remote") {
    action(type="omfile" dynaFile="RemotePerHostLogs"
           dirCreateMode="0755" fileCreateMode="0644")
}
EOF
```
Очищаем старые конфиги:
```
rm -f /etc/rsyslog.d/debug*.conf 2>/dev/null
rm -f /etc/rsyslog.d/backup 2>/dev/null
rm -f /etc/rsyslog.d/*.bak 2>/dev/null
rm -f /etc/rsyslog.d/*~ 2>/dev/null
```
Создаем директорию для логов:
```
mkdir -p /opt
chmod 755 /opt
```
Проверяем и запускаем rsyslog:
```
rsyslogd -N1
systemctl restart rsyslog
systemctl status rsyslog --no-pager -l
netstat -tulpn | grep 514
```

<img width="875" height="69" alt="image" src="https://github.com/user-attachments/assets/9638c587-5455-46e3-90aa-0c927c4c1ff0" />

### Настраиваем BR-RTR, BR-SRV, HQ-RTR:
### Возможно придется добавить правило в iptables на роутерах:
```
iptables -t nat -L PREROUTING -n -v | grep 514
iptables -I FORWARD -s 192.168.2.14 -d 192.168.1.62 -p upd --dport 514 -j ACCEPT
iptabes -I FORWARD -s 192.168.1.62 -d 192.168.2.14 -p udp --sport 514 -j ACCEPT
iptables -I FORWARD -p tcp --dport 514 -j ACCEPT
iptabes -I FORWARD -p udp --dport 514 -j ACCEPT
iptables-save > /etc/iptables/rules.v4
```

Устанавливаем rsyslog:
```
apt-get update && apt-get install rsyslog -y
```
Редачим конфиг:
```
cat > /etc/rsyslog.conf << 'EOF'
# rsyslog configuration file

#### MODULES ####
module(load="imuxsock")
module(load="imklog")

#### GLOBAL DIRECTIVES ####
global(workDirectory="/var/spool/rsyslog")

#### RULES ####
# Local logs
*.info;mail.none;authpriv.none;cron.none   /var/log/messages
authpriv.*   /var/log/secure
mail.*       /var/log/maillog
cron.*       /var/log/cron
*.emerg      :omusrmsg:*

# Send all logs to central server
*.* @@192.168.1.62:514
EOF
```
Проверяем и запускаем rsyslog:
```
rsyslogd -N1
systemctl restart rsyslog
systemctl status rsyslog --no-pager -l
```
Проверяем отправку:
```
logger -p user.warning "Test WARNING from $(hostname) $(date)"
logger -p user.crit "Test CRIT from $(hostname) $(date)"
logger -p user.info "Test INFO (SHOULD NOT ARRIVE) from $(hostname) $(date)"
```
### На сервере должна появится директория в /opt и там файлы с логами:

<img width="960" height="251" alt="image" src="https://github.com/user-attachments/assets/b9f5358f-72e5-4c68-81a3-6f1cafe623ef" />

Настраиваем ротацию:
```
cat > /etc/logrotate.d/central-logs << 'EOF'
/opt/*/*.log {
    weekly
    rotate 4
    size 10M
    compress
    delaycompress
    missingok
    notifempty
    create 0644 root root
    sharedscripts
    postrotate
        systemctl kill -s HUP rsyslog > /dev/null 2>&1 || true
    endscript
}
EOF
```
Проверяем конфиг:
```
logrotate -d /etc/logrotate.d/central-logs
logrotate -f /etc/logrotate.d/central-logs
```

<img width="1218" height="789" alt="image" src="https://github.com/user-attachments/assets/b8be42d8-c9ca-4d55-a902-26fff68bc3fc" />

<img width="596" height="838" alt="image" src="https://github.com/user-attachments/assets/73a9de9e-4ad9-46ef-9a3b-15eb100ab76e" />

Проверяем ротацию:
```
ls -la /opt/br-srv/
logrotate -f /etc/logrotate.d/central-logs
ls -la /opt/br-srv/*.gz 2>/dev/null
```
Добавляем в крон:
```
ls -la /etc/cron.daily/ | grep logrotate
echo "0 0 * * 0 /usr/sbin/logrotate /etc/logrotate.conf > /dev/null 2>&1" >> /var/spool/cron/root
crontab -l
```
### Проверяем все:
```
ls -la /opt/
ls -la /opt/br-srv/
```
На любом клиенте:
```
logger -p user.info "INFO message (SHOULD NOT ARRIVE)"
logger -p user.warning "WARNING message (SHOULD ARRIVE)"
logger -p user.err "ERR message (SHOULD ARRIVE)"
```
На сервере:
```
logrotate -f /etc/logrotate.d/central-logs
```

<img width="679" height="245" alt="image" src="https://github.com/user-attachments/assets/ca00cb7c-f6be-4298-9177-69cffd27bb89" />

 </details>

## На сервере HQ-SRV реализуйте мониторинг устройств с помощью открытого программного обеспечения

<details>
    <summary>ЗАДАНИЕ</summary>

Обеспечьте доступность по URL - http://mon.au-team.irpo для сетей офиса HQ, внесите изменения в инфраструктуру разрешения доменных имён 

• Мониторить нужно устройства HQ-SRV и BR-SRV 

• В мониторинге должны визуально отображаться нагрузка на ЦП, объем занятой ОП и основного накопителя 

• Логин и пароль для службы мониторинга admin P@ssw0rd 

• Организуйте доступ к мониторингу для HQ-CLI, без внешнего доступа 

• Выбор программного обеспечения, основание выбора и основные параметры с указанием порта, на котором работает мониторинг, отметьте в отчёте

 </details>

 <details>
    <summary>НАЖМИ</summary>

 Добавляем DNS записи на HQ-SRV и HQ-CLI:
HQ-SRV: 
```
echo "127.0.0.1 mon.au.team" >> /etc/hosts
```
HQ-CLI:
```
echo "192.168.1.62 mon.au.team" >> /etc/hosts
```
### Будем использовать prometheus с grafana:
### HQ-SRV:
Создаем пользователя для сервисов:
```
useradd --no-create-home --shell /bin/false prometheus
useradd --no-create-home --shell /bin/false node_exporter
```
Скачиваем Prometheus:
```
cd /tmp
curl -LO https://github.com/prometheus/prometheus/releases/download/v2.55.1/prometheus-2.55.1.linux-amd64.tar.gz
tar -xvf prometheus-2.55.1.linux-amd64.tar.gz
mv prometheus-2.55.1.linux-amd64 /opt/prometheus
```
Создаем директорию:
```
mkdir -p /opt/prometheus/data
chown -R prometheus:prometheus /opt/prometheus
```
Конфигурируем Prometheus:
```
cat > /opt/prometheus/prometheus.yml << 'EOF'
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'hq-srv'
    static_configs:
      - targets: ['192.168.1.62:9100']
        labels:
          instance: 'hq-srv'
          group: 'servers'

  - job_name: 'br-srv'
    static_configs:
      - targets: ['192.168.2.14:9100']
        labels:
          instance: 'br-srv'
          group: 'servers'
EOF

chown prometheus:prometheus /opt/prometheus/prometheus.yml
```
Создаем systemd юнит:
```
cat > /etc/systemd/system/prometheus.service << 'EOF'
[Unit]
Description=Prometheus
After=network.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/opt/prometheus/prometheus \
    --config.file=/opt/prometheus/prometheus.yml \
    --storage.tsdb.path=/opt/prometheus/data \
    --web.console.templates=/opt/prometheus/consoles \
    --web.console.libraries=/opt/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
EOF
```
Устанавливаем node_exporter:
```
cd /tmp
curl -LO https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
tar -xvf node_exporter-1.8.2.linux-amd64.tar.gz
mv node_exporter-1.8.2.linux-amd64 /opt/node_exporter
chown -R node_exporter:node_exporter /opt/node_exporter
```
Создаем systemd юнит для node_exporter:
```
cat > /etc/systemd/system/node_exporter.service << 'EOF'
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/opt/node_exporter/node_exporter

[Install]
WantedBy=multi-user.target
EOF
```
Запускаем Prometheus и Node_Exporter:
```
systemctl daemon-reload
systemctl enable prometheus node_exporter
systemctl start prometheus node_exporter
systemctl status prometheus --no-pager -l
systemctl status node_exporter --no-pager -l
netstat -tulpn | grep -E "9090|9100"
```
### BR-SRV:
Создаем пользователя:
```
useradd --no-create-home --shell /bin/false node_exporter
```
Устанавливаем node_exporter:
```
cd /tmp
curl -LO https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
tar -xvf node_exporter-1.8.2.linux-amd64.tar.gz
mv node_exporter-1.8.2.linux-amd64 /opt/node_exporter
chown -R node_exporter:node_exporter /opt/node_exporter
```
Создаем systemd юнит:
```
cat > /etc/systemd/system/node_exporter.service << 'EOF'
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/opt/node_exporter/node_exporter

[Install]
WantedBy=multi-user.target
EOF
```
Запускаем node_exporter:
```
systemctl daemon-reload
systemctl enable node_exporter
systemctl start node_exporter
systemctl status node_exporter --no-pager -l
netstat -tulpn | grep 9100
```
### HQ-SRV:
Устанавливаем Grafana:
```
apt-get update && apt-get install shadow-utils fontconfig -y
wget https://dl.grafana.com/oss/release/grafana-11.2.0.linux-amd64.tar.gz
tar -zxvf grafana-11.2.0.linux-amd64.tar.gz
mv grafana-v11.2.0 /opt/grafana
```
Создаем пользователя:
```
useradd --no-create-home --shell /bin/false grafana
chown -R grafana:grafana /opt/grafana
```
Создаем systemd юнит для grafana:
```
cat > /etc/systemd/system/grafana.service << 'EOF'
[Unit]
Description=Grafana
After=network.target

[Service]
User=grafana
Group=grafana
Type=simple
ExecStart=/opt/grafana/bin/grafana-server -homepath /opt/grafana

[Install]
WantedBy=multi-user.target
EOF
```
Конфигурируем grafana:
```
cat > /opt/grafana/conf/custom.ini << 'EOF'
[server]
http_port = 3000
domain = mon.au.team
root_url = http://mon.au.team:3000

[auth]
disable_login_form = false

[auth.anonymous]
enabled = false

[security]
admin_user = admin
admin_password = P@ssw0rd
EOF

chown grafana:grafana /opt/grafana/conf/custom.ini
```
Запускаем grafana:
```
systemctl daemon-reload
systemctl enable grafana
systemctl start grafana
systemctl status grafana --no-pager -l
netstat -tulpn | grep 3000
```
На HQ-CLI добавляем dns запись:
```
echo "192.168.1.62 mon.au.team" >> /etc/hosts
```

Открываем графану по адресу mon.au.team:3000
```
login:admin
password:P@ssw0rd
```
Заходим сюда:

<img width="1719" height="885" alt="image" src="https://github.com/user-attachments/assets/43b845cc-e553-4486-a2d9-929c64a529d2" />

Жмем Add data source и выбираем prometheus:

<img width="1716" height="923" alt="image" src="https://github.com/user-attachments/assets/9c8ca5af-de67-4fab-8bbb-3e9ad72b10e8" />

Вбиваем сюда http://localhost:9090

<img width="1710" height="855" alt="image" src="https://github.com/user-attachments/assets/213ed10d-702e-42c0-b7bb-408028809a8c" />

Внизу жмем save & test:

<img width="1078" height="243" alt="image" src="https://github.com/user-attachments/assets/b1d74848-0899-4fed-8193-62052b2d99ea" />

Жмем на + в правом верхнем углу, import dashboard:

<img width="1715" height="876" alt="image" src="https://github.com/user-attachments/assets/1de00880-1e70-4119-84ce-21268e878119" />

Вводим 1860 в поле и нажимаем Load:

<img width="1715" height="873" alt="image" src="https://github.com/user-attachments/assets/52dd0029-63ef-4904-8708-de47f8c535f4" />

Жмем import:

<img width="1716" height="875" alt="image" src="https://github.com/user-attachments/assets/12574dcf-98eb-4015-970c-bec7211e57ff" />

Должно получиться так:

<img width="1718" height="878" alt="image" src="https://github.com/user-attachments/assets/f8b9b3e5-09fa-4277-ae8d-356801821119" />

Настраиваем iptables на hq-rtr:
```
iptables -I INPUT -p tcp --dport 9090 -s 192.168.1.0/26 -j ACCEPT
iptables -I INPUT -p tcp --dport 9090 -j DROP
iptables -I INPUT -p tcp --dport 3000 -s 192.168.1.0/26 -j ACCEPT
iptables -I INPUT -p tcp --dport 3000 -j DROP
iptables -I INPUT -p tcp --dport 9100 -s 192.168.2.14 -j ACCEPT
iptables-save > /etc/iptables/rules.v4
```

 </details>

## Реализуйте механизм инвентаризации машин HQ-SRV и HQ-CLI через Ansible на BR-SRV

<details>
    <summary>ЗАДАНИЕ</summary>
 
Плейбук должен собирать информацию о рабочих местах: 

• Имя компьютера 

• IP-адрес компьютера 

• Плейбук, должен быть размещен в директории /etc/ansible, отчёты в поддиректории PC-INFO, в формате .yml. Файлы должны называется именем компьютера, который был инвентаризирован 

• Файл плейбука располагается в образе Additional.iso в директории playbook

 </details>

 <details>
    <summary>НАЖМИ</summary>

 Возможно придется прокинуть ключи для рута на машины по аналогии с настройкой ansible во 2 модуле:
 ```
ssh-keygen
ssh-copy-id -p 2026 sshuser@192.168.1.62
ssh-copy-id -p 2026 sshuser@192.168.1.3
ssh-copy-id -p 2026 sshuser@192.168.2.1
ssh-copy-id -p 2026 sshuser@192.168.1.1
```
Создаем плейбук:
```
vim /etc/ansible/inventory-pc.yml
```
Вставляем (тут важны пробелы в строчках, как файл должен выглядеть скрин ниже):
```
---
---
- name: Collect PC Information
  hosts: 192.168.1.3, 192.168.1.62
  gather_facts: yes
  tasks:
    - name: Create PC-INFO directory
      local_action: 
        module: file
        path: /etc/ansible/PC-INFO
        state: directory
        mode: '0755'

    - name: Generate PC report
      local_action:
        module: copy
        content: |
          hostname: {{ ansible_hostname }}
          ip_address: {{ ansible_default_ipv4.address }}
        dest: "/etc/ansible/PC-INFO/{{ ansible_hostname }}.yml"
```

<img width="1709" height="845" alt="image" src="https://github.com/user-attachments/assets/c3c5965f-be15-4c2d-8ce9-5695dbe17aef" />

<img width="628" height="431" alt="image" src="https://github.com/user-attachments/assets/a9835584-f5f2-4ece-97a6-a088aaa56a7e" />

Создаем директорию для отчетов:
```
mkdir -p /etc/ansible/PC-INFO
```
Запускаем:
```
cd /etc/ansible
ansible-playbook inventory-pc.yml
```
Получаем:

<img width="934" height="684" alt="image" src="https://github.com/user-attachments/assets/5beee67f-8573-44a0-a4cb-4570d54964b0" />

Проверяем:
```
ls -la /etc/ansible/PC-INFO/
cat /etc/ansible/PC-INFO/*.yml
```

 </details>

##  На HQ-SRV настройте программное обеспечение fail2ban для защиты ssh 

<details>
    <summary>ЗАДАНИЕ</summary>
 
Укажите порт ssh 

• При 3 неуспешных авторизациях адрес атакующего попадает в бан 

• Бан производится на 1минуту

 </details>

<details>
    <summary>НАЖМИ</summary>
 
Устанавливаем fail2ban:
```
apt-get update && apt-get install fail2ban -y
```
Создаем конфиг:
```
cat > /etc/fail2ban/jail.local << 'EOF'
[DEFAULT]
bantime = 60

findtime = 600

maxretry = 3

banaction = iptables-multiport
banaction_allports = iptables-allports

protocol = tcp

[sshd]
enabled = true
port = 2026
logpath = /var/log/secure
backend = systemd
EOF
```
Проверяем синтаксис и запускаем:
```
fail2ban-client -t
systemctl enable --now fail2ban
systemctl status fail2ban
```
Проверяем работоспособность:
```
fail2ban-client status sshd
```

<img width="621" height="181" alt="image" src="https://github.com/user-attachments/assets/2d3876a2-4f89-437c-94db-3b4bdfb79ab3" />

В конфиге ssh убираем ограничение на заход только через sshuser и убираем количество попыток входа /etc/openssh/sshd_config (да, это противоречит заданию из 1 модуля, но кого это волнует):
```
#AllowUsers sshuser
#MaxAuthTries 2
```
С другого компьютера пробуем неудачно зайти через ssh:
```
ssh -p 2026 user@192.168.1.62
```
На сервере проверяем что IP банится:
```
fail2ban-client status sshd
fail2ban-client banned
```
Снять бан можно командой:
```
fail2ban-client set sshd unbanip 192.168.2.14
```
 </details>
