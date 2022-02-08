## Домашняя работа к занятию 04.02.2022, VLAN и маршрутизация между VLAN 
### Задание:
1. В тестовой среде (EVE-NG) добавить объекты: роутер (R1), 2 коммутатора L2 (SW1,SW2), два компьютера (VPC1,VPC2) и объединить их сетью
2. Создать на устройствах VLANы и прописать их на портах коммутаторов
3. Настройить транки между коммутаторами
4. Настроить межвлановую маршрутизацию на роутерe
5. Проверить работоспособность межвлановой маршрутизации, используя компьютеры

#### Графическая схема


#### Таблица адресов и оборудования
| Устройстово | Интерфейс | IPv4 адрес | Маска |Шлюз по умолчанию |
|--- | --- | --- | --- | --- |
|R1|e0/0.3|192.168.3.1|255.255.255.0|---|
|R1|e0/0.4|192.68.4.1|255.255.255.0|---|
|R1|e0.0|---|---|---
|SW1|VLAN3|192.168.3.11|255.255.255.0|192.168.3.1
|SW2|VLAN3|192.168.3.12|255.255.255.0|192.168.3.1
|VPC1|eth0|192.168.3.3|255.255.255.0|192.168.3.1
|VPC2|eth0|192.168.4.3|255.255.255.0|192.168.4.1

#### Таблица VLAN
|VLAN|Название|Назначенные интерфейсы|
|--- | --- | --- |
|3|Managment|SW1:VLAN3 SW2:VLAN3  SW1:e0/2   |
|4|Operations|SW2:e0/1|
|7|ParkingLot|SW1:e0/3  SW2:e0/2-3|
|8|Native|---|


Конфиги оборудования

### R1
interface Ethernet0/0
 no ip address
!
interface Ethernet0/0.3
 description interface for vlan3
 encapsulation dot1Q 3
 ip address 192.168.3.1 255.255.255.0
!
interface Ethernet0/0.4
 description Interface for vlan4
 encapsulation dot1Q 4
 ip address 192.168.4.1 255.255.255.0
!
interface Ethernet0/0.8
 description Interface for native vlan8
 encapsulation dot1Q 8 native
!
interface Ethernet0/1
 no ip address
 shutdown
!
interface Ethernet0/2
 no ip address
 shutdown
!
interface Ethernet0/3
 no ip address
 shutdown

### SW1
interface Ethernet0/0
 switchport trunk allowed vlan 3,4,8
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 8
 switchport mode trunk
!
interface Ethernet0/1
 switchport trunk allowed vlan 3,4,8
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/2
 switchport access vlan 3
 switchport mode access
!
interface Ethernet0/3
 switchport access vlan 7
 switchport mode access
 shutdown
!
interface Vlan3
 ip address 192.168.3.11 255.255.255.0
!
ip default-gateway 192.168.3.1


### SW2
interface Ethernet0/0  
 switchport trunk allowed vlan 3,4,8  
 switchport trunk encapsulation dot1q  
 switchport mode trunk  
!
interface Ethernet0/1  
 switchport access vlan 4  
 switchport mode access  
!
interface Ethernet0/2
 switchport access vlan 7
 switchport mode access
 shutdown
!
interface Ethernet0/3
 switchport access vlan 7
 switchport mode access
 shutdown
!
interface Vlan3
 ip address 192.168.3.12 255.255.255.0
 shutdown
!
ip default-gateway 192.168.3.1

