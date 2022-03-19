## Домашняя работа к занятию 15.02.2022, DHCPv4/v6 и SLAAC 
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





4. Настройить и проверить работу двух DHCPv4 серверов на R1
5. Настроить и проверить DHCP Relay на R2
