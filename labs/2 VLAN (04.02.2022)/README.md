## Домашняя работа к занятию 04.02.2022, VLAN и маршрутизация между VLAN (Router on a Stick)
### Задание:
1. В тестовой среде (EVE-NG) добавить объекты: роутер (R1), 2 коммутатора L2 (SW1,SW2), два компьютера (VPC1,VPC2) и объединить их сетью
2. Создать на устройствах VLANы и прописать их на портах коммутаторов
3. Настроить транк между коммутаторами
4. Настроить межвлановую маршрутизацию на роутерe
5. Проверить работоспособность межвлановой маршрутизации, используя компьютеры

#### Графическая схема
![alt-текст](https://github.com/umostel/OTUShw/blob/main/labs/2%20VLAN%20(04.02.2022)/lab02.png "графическая схема к лабараторной работе")

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

## Выполнение работы
1. В виртуальной среде eve-ng добавляем объекты на рабочую область (R1, SW1, SW2, VPC1 и VPC2) и коммутируем их между собой в требуемые порты в соответсвии с таблицей. В качестве образа роутера R1 используем образ L3-adventerprisek9-m-15.4-2T.bin, в качестве образа коммутаторов SW1 и SW2 - образ L2-ADVENTERPRISEK9-M-15.2-20150703.bin, а для компьютера задействуем стандартный шаблон Virtual PC (VPCS). Проводим минимальную настроку окружения (имя хоста, отключаем резолвинг, включаем шифрование паролей, , баннер на вход) и авторизации (пароли на привелигированный режим, на консоль и терминал )  на всех устройств на базе Cisco IOS:
```
conf t
hostname R1
no ip domain-lookup
enable secret class
line console 0
password cisco
login
exit
line vty 0 4
password cisco
login
exit
service password-encryption
banner login Unathorization access deny
```
Приведенный выше образец - для R1. По аналогии делаем для SW1 и SW2

2.  Создаем на устройствах VLANы и прописываем их на портах коммутаторов.
Создаем VLAN и задаем их описание. Комманды для SW1 и SW2 - идентичные
```
vlan 3
name Managment
vlan 4 
name Operation
vlan 7
name ParkingLot
vlan 8
name Native
exit
```

В VLAN3 создаем SVI и прописываем шлюз по умолчанию.
```
interface vlan 3
ip address 192.168.3.11 255.255.255.0
no shutdown
exit
ip default-gateway 192.168.3.1
```
Пример выше длы SW1. Для SW2 используем IP 192.168.3.12/24


Неиспользуемые порты переводим в access режим и добавляем в VLAN7, порты на которых подключены VPC переводим в access режим и помещаем в VLAN3
```
interface e0/3
switchport mode access
switchport access vlan 7
shutdown
exit
interface e0/2
switchport mode access
switchport access vlan 3
no shutdown
exit
```
Пример выше длы SW1. Для SW2 делаем в соответсвии с таблицей VLAN.  
На каждом подэтапе пункта 3 контроллируем результат на коммутаторах командами:
```
show vlan breif
show ip interface breif
```
3. Настраиваем транк между коммутаторами и между роутером и коммутатором со стороны коммутатора. Тип инкапусляции dot1q, разрешаем хождение вланов 3,4,8(нативный). 
```
interface e0/0
switchport trunk encapsulation dot1q
switchport mode trunk
switchport allowed vlan 3,4,8
exit
```
Пример выше длы SW1 в сторону SW2. Для SW2 в сторону SW1 и SW1 в сторону R1  аналогично, но для соответсвующих интерфейсов согласно схеме.
Контроллируем результат на каждом коммутаторе командой:
```
show interfaces trunk
```

4. Настроиваем межвлановую маршрутизацию на роутерe.
Создаем и настраиваем субинтерфейсы, в соответсвии с таблицей IP адресов, к соответсвующим VLAN приходящим из транка на интерфейс e0/0 со стороны SW1 (3,4,8). Vlan8 нативный, для него адресацию не задаем.

```
interface e0/0.3
encapsulation dot1q 3
ip address 192.168.3.1 255.255.255.0
exit
interface e0/0.4
encapsulation dot1q 4
ip address 192.168.4.1 255.255.255.0
interface e0/0.8
encapsulation dot1q 8 native
exit
```
Контроллируем результат командами:
```
show ip interface breif
show ip route
show interfaces trunk
```
5.  Проверяем работоспособность, используя компьютеры VPC1(vlan3) и VPC2 (vlan4)
Назначаем ip адреса и указываем шлюз по умолчанию на интерфейсы VPC.  
Для VPC1:
```
ip 192.168.3.3/24 192.168.3.1
```
Для VPC2:
```
ip 192.168.4.3/24 192.168.4.1
```
"Пингуем" с VPC1->VPC2 и с VPC2->VPC1 . Если ответ получен, то все настроено верно.



### Итоговые конфигурации оборудования (сокращенный вид)

### R1
```
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
```



### SW1
```
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
```

### SW2
```
interface Ethernet0/0  
 switchport trunk allowed vlan 3,4,8  
 switchport trunk encapsulation dot1q  
 switchport trunk native vlan 8
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
```

[Полная хронология комманд для R1](https://github.com/umostel/OTUShw/blob/main/labs/2%20VLAN%20(04.02.2022)/configs/R1)
