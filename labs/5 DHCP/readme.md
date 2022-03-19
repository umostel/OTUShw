## Домашняя работа к занятию 15.02.2022, DHCPv4/v6 и SLAAC 
#### I. IPv4
### Задание:

1. В тестовой среде (EVE-NG) добавить объекты: два компьютера (VPC-A,VPC-B), два коммутатора L2 (SW1,SW2), два роутера (R1,R2) и объединить согласно схеме
2. Рассчитать схему адресов и внести данные в таблицу адресов оборудования
3. Произвести настройку интерфейсов и VLAN устройств согласно таблиц
4. Настройить и проверить работу двух DHCPv4 серверов на R1
5. Настроить и проверить DHCP Relay на R2

#### Графическая схема
![alt-текст](https://github.com/umostel/OTUShw/blob/main/labs/5%20DHCP/labDHCPv4.jpg "графическая схема к лабараторной работе DHCPv4")

#### Таблица адресов и оборудования
| Устройстово | Интерфейс | IPv4 адрес | Маска |Шлюз по умолчанию |
|--- | --- | --- | --- | --- |
|R1|e0/1|10.0.0.1|255.255.255.252|---|
|R1|e0/0|---|---|---|
|R1|e0/0.100|---|---|---|
|R1|e0/0.200|---|---|---|
|R1|e0/0.1000|---|---|---|
|R2|e0/1|10.0.0.2|255.255.255.252|---|
|R2|e0/0|---|---|---|
|SW1|VLAN200|---|---|---|
|SW2|VLAN1||---|---|---|
|VPC-A|eth0|DHCP|DHCP|DHCP|
|VPC-B|eth0|DHCP|DHCP|DHCP|

#### Таблица VLAN
|VLAN|Название|Назначенные интерфейсы|
|--- | --- | --- |
|1|---|SW2:e0/1|
|100|Clients|SW1:e0/1|
|200|Managment|SW1:VLAN200|
|999|Parking_Lot|SW1:e0/2-3,SW2:e0/2-3,!!!!!!!!!!!!!!!!!|
|1000|Native|---|

## Выполнение работы.
1. В виртуальной среде eve-ng добавляем объекты на рабочую область (R1, R2, SW1, SW2, VPC-A и VPC-B) и коммутируем их между собой в требуемые порты в соответсвии с графической схемой. В качестве образа роутеров R1 и R2 используем образ L3-adventerprisek9-m-15.4-2T.bin, в качестве образа коммутаторов SW1 и SW2 - образ L2-ADVENTERPRISEK9-M-15.2-20150703.bin, а для компьютеров задействуем стандартный шаблон Virtual PC (VPCS). Проводим минимальную базовую настроку окружения (имя хоста, отключаем резолвинг, включаем шифрование паролей, баннер на вход, запрет вывода консольных сообщений в процессе конфигурирования) и авторизации (пароли на привелигированный режим, на консоль и терминал )  на всех устройств на базе Cisco IOS:
```
enable
conf t
hostname SW1
no ip domain-lookup
banner login %
####################################
#                                  #
#     Unauthorized access deny!!   #
#                                  #
####################################
%
line console 0
logging synchronius
password cisco
login
exit
line vty 0 4
logging synchronous 
password cisco
login
exit
enable password class
service password-encryption 
```
Комманды выше для SW1, аналогично делаем на R1,R2,SW2
2. Делим сеть 192.168.1.0.24 на подсети и расчитываем схему адресации, согласно задания:
2.1. ПодсетьA 192.168.1.0/26 для VLAN100 (Clients) на R1
Первый адрес диапазона на R1:e0/0.100 - 192.168.1.1
2.2. ПодсетьB 192.168.1.64/27 для VLAN200 (Management)на R1
Первый адрес диапазона на R1:e0/0.200 - 192.168.1.65
Второй адрес диапазона на SW1:VLAN200 - 192.168.1.66
2.3. ПодсетьC 192.168.1.96/28 для пользователей R2
Первый адрес диапазона на R2:e0/0 - 192.168.1.97

Вносим эти данные в таблицу адресов:
#### Таблица адресов и оборудования
| Устройстово | Интерфейс | IPv4 адрес | Маска |Шлюз по умолчанию |
|--- | --- | --- | --- | --- |
|R1|e0/1|10.0.0.1|255.255.255.252|---|
|R1|e0/0|---|---|---|
|R1|e0/0.100|192.168.1.1|255.255.255.192|---|
|R1|e0/0.200|192.168.1.65|255.255.255.224|---|
|R1|e0/0.1000|---|---|---|
|R2|e0/1|10.0.0.2|255.255.255.252|---|
|R2|e0/0|192.168.1.97|255.255.255.240|---|
|SW1|VLAN200|192.168.1.66|255.255.255.224|---|
|SW2|VLAN1||---|---|---|
|VPC-A|eth0|DHCP|DHCP|DHCP|
|VPC-B|eth0|DHCP|DHCP|DHCP|

2. Производим настройку интерфейсов и VLAN устройств согласно схемы
На R1
```
R1(config)# int e0/1
R1(config-if)# ip address 10.0.0.1 255.255.255.252
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)#int e0/0.100
R1(config-subif)#ip address 192.168.1.1 255.255.255.192
R1(config-subif)#no shut
R1(config-subif)#exit
R1(config)#int e0/0.200
R1(config-subif)#ip address 192.168.1.65 255.255.255.224
R1(config-subif)#no shut
R1(config-subif)#exit
```

На SW1
```
SW1(config)#vlan 100
SW1(config-vlan)#name Clients
SW1(config-vlan)#exit
SW1(config)#vlan 200
SW1(config-vlan)#name Managment
SW1(config-vlan)#exit
SW1(config)#Vlan 999
SW1(config-vlan)#name Parking_lot
SW1(config-vlan)#exit
SW1(config)#
SW1(config)#int vlan 200
SW1(config-if)#ip address 192.168.1.66 255.255.255.224
SW1(config-if)#no shut
SW1(config-if)#exit
SW1(config)#int e0/1
SW1(config-if)#switchport mode access
SW1(config-if)#switchport access vlan 100
SW1(config-if)#exitint e0
SW1(config)#int e0/0
SW1(config-if)#switchport trunk encapsulation dot1q
SW1(config-if)#switchport mode trunk
SW1(config-if)#switchport trunk allowed vlan 100,200
SW1(config-if)#switchport trunk native vlan 1000
SW1(config-if)#exit
SW1(config)#int range e0/2-3
SW1(config-if-range)#switchport mode access
SW1(config-if-range)#switchport access vlan 999
SW1(config-if-range)#exit
```

На R2
```
R2(config)#int e0/1
R2(config-if)#ip address 10.0.0.2 255.255.255.252
R2(config-if)#no shut
R2(config-if)#exit
R2(config)#int e0/0
R2(config-if)#ip add 192.168.1.97 255.255.255.240
R2(config-if)#no shut
R2(config-if)#exit
```

Прописываем маршруты по умолчанию на R1 и R2, указывающие на адрес ip интерфейса e0/1 R2 и R1 соответсвенно:
На R1
```
R1(config)#ip route 0.0.0.0 0.0.0.0 10.0.0.2
```
На R2
```
#R2(config)#ip route 0.0.0.0 0.0.0.0 10.0.0.1
```
Проверяем сделанные настройки:  
с R1 пингуем ip интерфейса R2:e0/0 R2 (192.168.1.97)
```
R1#ping 192.168.1.97
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.97, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```
с R2 пингуем ip интерфейса R1:e0/0.100 (192.168.1.1) и ip интерфейса R2:e0/0.200 (192.168.1.65)
```
R2#ping 192.168.1.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
R2#ping 192.168.1.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
R2#ping 192.168.1.65
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.65, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```

В случае проблем, для траблшутинга используем комманды:
show ip interfaces brief
show interfaces status
show interfaces trunk
show vlan 
show vlan breif
show ip route

4. Настройить и проверить работу двух DHCPv4 серверов на R1
Настраиваем DHCP сервер для двух подсетей (А и С). На R1 исключаем пять первых адресов из каждого пула и создаем два пула (SubA и SubC): указываем сети, которые используют эти пулы;  имя домена otus-lab.local; соответсвующие шлюзы по умолчанию; время аренды 2д12ч30м.

```
R1(config)#ip dhcp excluded-address 192.168.1.1 192.168.1.5
R1(config)#ip dhcp pool SubA
R1(dhcp-config)#network 192.168.1.0 255.255.255.192
R1(dhcp-config)#domain-name otus-lab.local
R1(dhcp-config)#default-router 192.168.1.1
R1(dhcp-config)#lease 2 12 30
R1(dhcp-config)#exit
R1(config)#ip dhcp excluded-address 192.168.1.97 192.168.1.98
R1(config)#ip dhcp pool SubB
R1(dhcp-config)#network 192.168.1.96 255.255.255.240
R1(dhcp-config)#domain-name otus-lab.local
R1(dhcp-config)#default-router 192.168.1.97
R1(dhcp-config)#lease 2 12 30
R1(dhcp-config)#exit
```
Проверяем работсопособность DHCP, пытаясь получить адрес на VPC-A и проверка icmp доступность шлюза.
```
VPCS> ip dhcp
DDORA IP 192.168.1.6/26 GW 192.168.1.1
VPCS> ping 192.168.1.1
84 bytes from 192.168.1.1 icmp_seq=1 ttl=255 time=0.269 ms
84 bytes from 192.168.1.1 icmp_seq=2 ttl=255 time=0.477 ms
84 bytes from 192.168.1.1 icmp_seq=3 ttl=255 time=0.551 ms
84 bytes from 192.168.1.1 icmp_seq=4 ttl=255 time=0.443 ms
84 bytes from 192.168.1.1 icmp_seq=5 ttl=255 time=0.469 ms
```
6. Настройка и проверка DHCP Relay на R2
На R2:e0/0 указываем ip R1:e0/1 как адрес для пересылки запросов dhcp 
```
R2(config)#int e0/0
R2(config-if)#ip helper-address 10.0.0.1
R2(config-if)
```
Проверяем работсопособность пересылки, пытаясь получить адрес на VPC-B и проверка icmp доступность шлюза и VPC-A.
``
VPCS> ip dhcp
DDORA IP 192.168.1.99/28 GW 192.168.1.97
VPCS> ping 192.168.1.97
84 bytes from 192.168.1.97 icmp_seq=1 ttl=255 time=0.247 ms
84 bytes from 192.168.1.97 icmp_seq=2 ttl=255 time=0.443 ms
^C
VPCS> ping 192.168.1.6
84 bytes from 192.168.1.6 icmp_seq=1 ttl=62 time=1.832 ms
84 bytes from 192.168.1.6 icmp_seq=2 ttl=62 time=0.736 ms
^C
```
Все работает.
В случае проблем, проводим диагностику и траблшутинг коммандами:
show ip dhcp pool 
show ip dhcp binding 
show ip dhcp server statistics 

R2#sh ip int e0/0
Ethernet0/0 is up, line protocol is up
  Internet address is 192.168.1.97/28
  Broadcast address is 255.255.255.255
  Address determined by setup command
  MTU is 1500 bytes
  Helper address is 10.0.0.1
```

#### II. IPv6. SLAAC и DHCPv6
### Задание:
Part 1: Build the Network and Configure Basic Device Settings
Part 2: Verify SLAAC address assignment from R1
Part 3: Configure and verify a Stateless DHCPv6 Server on R1
Part 4: Configure and verify a Stateful DHCPv6 Server on R1
Part 5: Configure and verify

#### Графическая схема
![alt-текст](https://github.com/umostel/OTUShw/blob/main/labs/5%20DHCP/labDHCPv6.jpg "графическая схема к лабараторной работе DHCPv6")

#### Таблица адресов и оборудования
| Устройстово | Интерфейс | IPv6 адрес | 
|--- | --- | --- |
|R3|e0/1|2001:db8:acad:2::1/64|
|R3|e0/1|fe80::1|
|R3|e0/0|2001:db8:acad:1::1/64|
|R3|e0/0|fe80::1|
|R3|e0/1|2001:db8:acad:2::2/64|
|R3|e0/1|fe80::2|
|R3|e0/0|2001:db8:acad:3::1/64|
|R3|e0/0|fe80::1|
|VPC-C|eth0|DHCP|
|VPC-D|eth0|DHCP|