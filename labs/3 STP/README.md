## Домашняя работа к занятию 08.02.2022, Избыточность локальных сетей. STP
### Задание:
1. В тестовой среде (EVE-NG) добавить объекты: 3 коммутатора L2 (SW1,SW2,SW3), объединить их сетью и произвести основные настройки параметров устройств


2. Выбор корневого моста
3. Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов
4. Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов


#### Графическая схема
![alt-текст](https://github.com/umostel/OTUShw/blob/main/labs/3%20STP/lab03.png "Графическая схема к лабораторной работе")

#### Таблица адресов и оборудования
| Устройство | Интерфейс | IPv4 адрес | Маска |Шлюз по умолчанию |
|--- | --- | --- | --- | --- |
|SW1|VLAN1|192.168.1.1|255.255.255.0|---|
|SW1|VLAN1|192.168.1.2|255.255.255.0|---|
|SW1|VLAN1|192.168.1.3|255.255.255.0|---|

#### Таблица VLAN
|VLAN|Название|Назначенные интерфейсы|
|--- | --- | --- |
|1|VLAN1|SW1:VLAN1, SW2:VLAN1, SW3:VLAN1|


## Выполнение работы
1. В виртуальной среде eve-ng добавляем три коммутатора на рабочую область (SW1, SW2, SW3) и коммутируем их между собой в требуемые порты в соответсвии с таблицей. В качестве образа коммутаторов SW1,  и SW2 - образ L2-ADVENTERPRISEK9-M-15.2-20150703.bin. Проводим минимальную настроку окружения (имя хоста, отключаем резолвинг, включаем шифрование паролей, баннер на вход, запрет вывода консольных сообщений в процессе конфигурирования) и авторизации (пароли на привелигированный режим, на консоль и терминал )  на всех устройств на базе Cisco IOS:
```
enable
conf t
hostname SW1
no ip domain-lookup
banner login %
Warning!
Authorized person only.
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
Настраиваем SVI VLAN, прописывая ip адреса согласно таблице. Для SW1 - 192.168.1.1/24
```
interface vlan1
ip address 192.168.1.1 255.255.255.0
no shudown
exit
```
Приведенный выше комманды - для SW1. По аналогии делаем для SW2 и SW3

Проверяем корректность сетевых настроек, коммандами show ip interface breif
Проверяем с каждого коммутатора доступность двух соседних коммутаторов:
```
SW1#ping 192.168.1.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
SW1#ping 192.168.1.3
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
````
Пример комманд и ответов для SW1. Делаем по аналогии для SW2 и SW3

2.  Что делаем
2.1 Отключаем все порты на коммутаторах, выполнив на каждом из них комманды:
```
interface range e0/0-3
shutdown
```
2.2. Переводим все порты на всех коммутаторах в режим trunk с инкапусляцией dot1q:
```
interface range e0/0-3
switchport trunk encapsulation dot1q
switchport mode trunk
exit
```
2.3. Включаем порты e0/1 и e0/3 на всех коммутаторах:
```
interface range e0/1, e0/3
no shutdown
exit
```
2.4. Коммандой show spannig-tree, отображаем данные протокола STP на кажом коммутаторе и определяем, какой коммутатор получил статус Root.
Вывод комманды для коммутатора SW1
```
SW1#show spanning-tree 

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.1000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Desg FWD 100       128.2    Shr 
Et0/3               Desg FWD 100       128.4    Shr 
```
Вывод комманды для коммутатора SW2
```
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        2 (Ethernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.2000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Root FWD 100       128.2    Shr 
Et0/3               Desg FWD 100       128.4    Shr 

```
Вывод комманды для коммутатора SW3
```
SW3#show spanning-tree 

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        4 (Ethernet0/3)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Altn BLK 100       128.2    Shr 
Et0/3               Root FWD 100       128.4    Shr 
```
Видео, что коммутатор SW1 выбран Root, поскольку его MAC адрес имеет наименьшее значение (aabb.cc00.1000 - SW1,aabb.cc00.2000 - SW2, aabb.cc00.3000 - SW3)  при прочих равных (Priority=Приоритет по умолчанию 32758 + 1 (VLAN)=32769)

3. Что делаем
```
Комманды
```

4. Что делаем
```
Комманды
```
5.  Что делаем
```
Комманды
```



### Итоговые конфигурации оборудования (сокращенный вид)

### R1
```
```



### SW1
```
```

### SW2
```
```
[Полная хронология комманд для R1](https://_______/configs/R1)  
[Полная хронология комманд для SW1]( ](https://_______//configs/SW1)  
[Полная хронология комманд для SW2](https:// ](https://_______/configs/SW2)  
[Полная хронология комманд для VPC1](https:// ](https://_______/configs/VPC1)  
[Полная хронология комманд для VPC2](https://github.com/](https://_______/configs/VPC2)  
