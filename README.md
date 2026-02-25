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
