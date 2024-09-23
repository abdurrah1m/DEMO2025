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

| Устройство | IP | Default gateway |
|:-:|:-:|:-:|
| ISP | 10.10.201.37/24 | 10.10.201.254 |
| | HQ 172.16.4.1/28 |  |
| | BR 172.16.5.1/28 |  |

# HQ-RTR

## Создание интерфейса, в сторону ISP

Назначим ip-адрес
```
interface ge0
description "ISP"
ip address 172.16.5.2/28
exit
```

Создаем service-instance с именем ge0/ge0
```
port ge0
service-instance ge0/ge0
encapsulation untagged
connect ip interface ge0
exit
exit
```

## Команды для проверки
```
show port
```
```
show port brief
```
```

```
