![image](https://github.com/user-attachments/assets/5e93864e-93db-42b9-ac98-494d5679c599)

| Машина | RAM, ГБ | CPU | HDD/SSD, ГБ | OS |
|:-:|:-:|:-:|:-:|:-:|
| ISP | 1 | 1 | 10 | ОС Альт JeOS/Linux или аналог |
| HQ-RTR | 1 | 1 | 10 | ОС EcoRouter или аналог |
| BR-RTR | 1 | 1 | 10 | ОС EcoRouter или аналог |
| HQ-SRV | 2 | 1 | 10 | ОС Альт Сервер или аналог |
| BR-SRV | 2 | 1 | 10 | ОС Альт Сервер или аналог |
| HQ-CLI | 3 | 2 | 15 | ОС Альт Рабочая Станция или аналог |
| Итого | 10 | 7 | 65 | - |

# 1. Произведите базовую настройку устройств

<details>
  <summary>1. Настройте имена устройств согласно топологии. Используйте полное доменное имя</summary>

## EcoRouter
```
en
```
```
conf t
```
```
hostname hq-rtr.au-team.irpo
```
```
hostname br-rtr.au-team.irpo
```
## Linux
```
hostnamectl set-hostname hq-srv.au.team.irpo;exec bash
```
```
hostnamectl set-hostname hq-cli.au.team.irpo;exec bash
```
```
hostnamectl set-hostname br-srv.au.team.irpo;exec bash
```

</details>

<details>
  <summary>2. На всех устройствах необходимо сконфигурировать IPv4</summary>

## EcoRouter
```
interface ge0
description "ISP"
ip address 172.16.4.2/28
exit
```

```
port ge0
service-instance ge0/ge0
encapsulation untagged 
connect ip interface ge0
exit
exit
```

Default gateway
```
ip route 0.0.0.0/0 172.16.4.1
```

Сохраняемся
```
write
```

Инфа по IP
```
show ip interface brief
```

</details>

<details>
  <summary>3. IP-адрес должен быть из приватного диапазона, в случае, если сеть локальная, согласно RFC1918</summary>

![image](https://github.com/user-attachments/assets/8e2832ff-9a62-4200-a68c-83b7a20af8d7)

</details>

<details>
  <summary>4. Локальная сеть в сторону HQ-SRV(VLAN100) должна вмещать не более 64 адресов</summary>

/26 255.255.255.192

</details>

<details>
  <summary>5. Локальная сеть в сторону HQ-CLI(VLAN200) должна вмещать не более 16 адресов</summary>

/28 255.255.255.240

</details>

<details>
  <summary>6. Локальная сеть в сторону BR-SRV должна вмещать не более 32 адресов</summary>

/27 255.255.255.224

</details>

<details>
  <summary>7. Локальная сеть для управления(VLAN999) должна вмещать не более 8 адресов</summary>

/29 255.255.255.248

</details>

<details>
  <summary>8. Сведения об адресах занесите в отчёт, в качестве примера используйте Таблицу 3</summary>

|Устройство|IP|Default gateway|
|:-:|:-:|:-:|
| ISP | DHCP 10.10.201.37/24 | 10.10.201.254 |
| ISP | HQ 172.16.4.1/28 | - |
| ISP | BR 172.16.5.1/28 | - |
| HQ-RTR | ISP 172.16.4.2/28 | 172.16.4.1 |
| HQ-RTR | SRV 10.10.10.1/26 | - |
| HQ-RTR | CLI 10.10.20.1/28 | - |
| BR-RTR | ISP 172.16.5.2/28 | 172.16.5.1 |
| BR-RTR | SRV 10.10.30.1/27 | - |
| HQ-SRV | 10.10.10.2/26 | 10.10.10.1 |
| HQ-CLI | 10.10.20.2/28 | 10.10.20.1 |
| BR-SRV | 10.10.30.2/27 | 10.10.30.1 |

</details>

# 2. Настройка ISP

Настройте адресацию на интерфейсах  

<details>
  <summary>Интерфейс, подключенный к магистральному провайдеру, получает адрес по DHCP</summary>
  
```
nano /etc/net/ifaces/ens18/options
```
```
BOOTPROTO=dhcp
```

</details>

<details>
  <summary>Настройте маршруты по умолчанию там, где это необходимо</summary>



</details>

<details>
  <summary>Интерфейс, к которому подключен HQ-RTR, подключен к сети 172.16.4.0/28</summary>

```
nano /etc/net/ifaces/ens19/options
```
```
BOOTPROTO=static
TYPE=eth
NM_CONTROLLED=no
DISABLED=no
CONFIG_IPV4=yes
```
```
nano /etc/net/ifaces/ens19/ipv4address
```
```
172.16.4.1/28
```

</details>

<details>
  <summary>Интерфейс, к которому подключен BR-RTR, подключен к сети 172.16.5.0/28</summary>



</details>

<details>
  <summary>На ISP настройте динамическую сетевую трансляцию в сторону HQ-RTR и BR-RTR для доступа к сети Интернет</summary>

Отключить NetworkManager:
```
systemctl disable NetworkManager
```
Настройки интерфейсов должны быть такими:
```
...
NM_CONTROLLED=no
DISABLED=no
...
```
Установка firewalld:
```
apt-get update && apt-get -y install firewalld && systemctl enable --now firewalld
```
Правила к исходящим пакетам (в сторону провайдера):
```
firewall-cmd --permanent --zone=public --add-interface=ens18
```
Правила к входящим пакетам (к локальной сети):
```
firewall-cmd --permanent --zone=trusted --add-interface=ens19
```
```
firewall-cmd --permanent --zone=trusted --add-interface=ens20
```
Включение NAT:
```
firewall-cmd --permanent --zone=public --add-masquerade
```
Сохранение правил:
```
firewall-cmd --complete-reload
```

</details>
