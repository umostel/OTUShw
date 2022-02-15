## Домашняя работа к занятию 08.02.2022, Избыточность локальных сетей. STP
### Задание:
1. В тестовой среде (EVE-NG) добавить объекты: 3 коммутатора L2 (SW1,SW2,SW3), объединить их сетью и произвести основные настройки параметров устройств
2. Выбор корневого моста
3. Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов
4. Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов
5. Ответить на итоговые вопросы


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

2.  Выбор корневого моста  
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
Вывод комманды для коммутатора SW1:
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
Вывод комманды для коммутатора SW2:
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
Вывод комманды для коммутатора SW3:
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
Видно, что коммутатор SW1 выбран Root bridge, поскольку его BID наименьший - MAC адрес имеет наименьшее значение (SW1: aabb.cc00.1000, SW2:aabb.cc00.2000, SW3:aabb.cc00.3000)  при прочих равных (Priority=Приоритет по умолчанию 32758 + 1 (VLAN)=32769)

## 2.5. Вопрос-ответ:
Вопрос 1: Какой коммутатор является корневым мостом?  
Ответ - SW1  
Вопрос 2: Почему этот коммутатор был выбран протоколом spanning-tree в качестве корневого моста?  
Ответ:У него наименьший BID среди всех коммутаторах: мак адрес наименьший, при равном приоритете (32769) и № VLAN (1).  
Вопрос 3: Какие порты на коммутаторе являются корневыми портами?  
Ответ: На SW2 – E0/1, на SW3-E0/3  
Вопрос 4: Какие порты на коммутаторе являются назначенными портами?  
Ответ: На SW1-E0/0 и E0/3, на SW2 – E0/3  
Вопрос 5: Какой порт отображается в качестве альтернативного и в настоящее время заблокирован?  
Ответ: На SW3 - E0/3  
Вопрос 6: Почему протокол spanning-tree выбрал этот порт в качестве невыделенного (заблокированного) порта?  
Ответ: Выбор был между портами SW2:E0/3 и SW3: E0/3. У SW3 – при равных стоимости корневых портов (100), BID больше (т.к. больше mac),  соответственно альтернативным выбран порт на SW3.  

3. Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов  
3.1. Отображаем состояние STP на некорневых коммутаторах SW2 и SW3:

Для SW2:
```
SW2# show spanning-tree
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
Для SW3:
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
Видим, что альтернативным портом, выбран порт E0/1 на коммутаторе SW3 

3.2. Уменьшаем стоимость корневого порта на коммутаторе SW3 на единицу:
```
SW3#: conf t
interface e0/3
spanning-tree cost 99
```

3.3. Наблюдаем  и анализируем изменение топологии STP:
На SW2
```
SW2#: show spanning-tree
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
             Aging Time  15  sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Root FWD 100       128.2    Shr
Et0/3               Altn BLK 100       128.4    Shr
```
На SW3
```
SW3#: show spanning-tree
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        99
             Port        4 (Ethernet0/3)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  15  sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Desg FWD 100       128.2    Shr
Et0/3               Root FWD 99        128.4    Shr
```
Наблюдаем, что альтернативный и выделенный порт поменялись местами.  

Вопрос: Почему протокол spanning-tree заменяет ранее заблокированный порт на назначенный порт и блокирует порт, который был назначенным портом на другом коммутаторе?  
Ответ: Сработало правило, что  выделенным портом выбирается порт с наименьшей стоимостью корневого порта.

3.4. Отменяем изменения стоимость корневого порта на коммутаторе SW3:
```
SW3#: conf t
interface e0/3
no spanning-tree cost
```
Выполняя команду show spanning-tree на SW2 и SW3 удостоверияемся, что состояние STP вернулось к выводу в п.3.1.

4. Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов
4.1. Включаем порты e0/0 и e0/2 на всех коммутаторах коммандами:
```
conf t
interface range e0/0, e0/2
no shutdown
```
Отображаем состояние STP на некорневых коммутаторах:

На SW2:
```
SW2#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        1 (Ethernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.2000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Root FWD 100       128.1    Shr
Et0/1               Altn BLK 100       128.2    Shr
Et0/2               Desg FWD 100       128.3    Shr
Et0/3               Desg FWD 100       128.4    Shr
```
На SW3:
```
SW3#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        3 (Ethernet0/2)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Altn BLK 100       128.1    Shr
Et0/1               Altn BLK 100       128.2    Shr
Et0/2               Root FWD 100       128.3    Shr
Et0/3               Altn BLK 100       128.4    Shr
```
Видим, что порт корневого моста переместился на порт с меньшим номером, связанный с коммутатором корневого моста, и перевел в альтернативное состояние предыдущий порт корневого моста.
# 4.2. Вопрос-ответ:
Вопрос 1: Какой порт выбран протоколом STP в качестве порта корневого моста на каждом коммутаторе некорневого моста?  
Ответ: На SW2 – E0/0, на SW3-E0/2  
Вопрос 2: Почему протокол STP выбрал эти порты в качестве портов корневого моста на этих коммутаторах?  
Ответ: На SW2 выбор был между портами E0/0 и E0/1. При равных cтоимости и BID, сделан на основании наименьшего приоритета портов, у E0/0=128.1, у  E0/1=128.2. Аналогично выбор сделан и на SW3 между портами E0/2 и E0/3  

# 5. Итоговые вопросы:
Вопрос 1.	Какое значение протокол STP использует первым после выбора корневого моста, чтобы определить выбор порта?
Ответ: Стоимость порта
Вопрос 2.	Если первое значение на двух портах одинаково, какое следующее значение будет использовать протокол STP при выборе порта? 
Ответ: BID (priority+mac+vlan)
Вопрос 3.	Если оба значения на двух портах равны, каким будет следующее значение, которое использует протокол STP при выборе порта? 
Ответ: идентификатор порта



