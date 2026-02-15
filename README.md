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
