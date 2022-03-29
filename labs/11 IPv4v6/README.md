### IPv4/6
#### Задание
1. Распланировать адресное пространство используя ipv4 и ipv6
2. Задокументировать адресное пространство
3. Настроить IP на всех активных портах

### Выполнение работы.
#### 1. Распланировать адресное пространство
Общий принцип планирования:  
##### для IPv4:  
10.10.1.Х/24 - подсеть для управляющей VLAN777, где Х - номер устройства;  
192.168.Х.Y/24 - подсеть для конечных устройств, где Х - номер VLAN равный номеру конечного VPC;  
172.16.X.Y/30 - для связи между роутерами внутри одной AS или сайта, где Х зависит от AS (Х=254 (AS1001), Х=52 (AS520), Х=42 (AS2042));  
195.178.0.Х/30 - для связи между роутерами из разных AS или сайтов.  

##### для IPv6:
fc00:777/32 - подсеть для управляющей VLAN777, последний блок в адресе - номер устройства;  
2003:0:0:X/64 - подсеть для конечных устройств, где Х - номер VLAN равный номеру конечного VPC;  
2001:db8:Х:Y/64 - для связи между роутерами внутри одной AS или сайта. где Х равен номеру AS, Y - номер по порядку, по количеству соединений. Последний блок в адресе - номер устройства;  
2002:db8:X:Y/64 - для связи между роутерами из разных AS или сайтов. X, Y - номера AS или роутеров между которыми соединение. Последний блок в адресе - номер устройства.  

 172.16.254.X/30 , 2001:db8:1001/64 - связь между роутерами внутри AS1001 
| **IPv4 Subnet**  | **IPv6 Subnet**    | **Назначение** |
|------------------|--------------------|----------------|
| 172.16.254.0/30  | 2001:db8:1001:1/64 | R12-R14        |
| 172.16.254.4/30  | 2001:db8:1001:2/64 | R12-R15        |
| 172.16.254.8/30  | 2001:db8:1001:3/64 | R13-R15        |
| 172.16.254.12/30 | 2001:db8:1001:4/64 | R13-R14        |
| 172.16.254.16/30 | 2001:db8:1001:5/64 | R19-R14        |

172.16.52.X/31 , 2001:db8:520/64 - связь между роутерами внутри AS520  
| **IPv4 Subnet** | **IPv6 Subnet**   | **Назначение** |
|-----------------|-------------------|----------------|
| 172.16.52.0/32  | 2001:db8:520:1/64 | R23-R25        |
| 172.16.52.4/32  | 2001:db8:520:2/64 | R23-R24        |
| 172.16.52.8/32  | 2001:db8:520:3/64 | R24-R26        |
| 172.16.52.12/32 | 2001:db8:520:4/64 | R25-R26        |

172.16.42.X/31 и 2001:db8:2042 - связь между роутерами внутри AS2042
| **IPv4 Subnet** | **IPv6 Subnet**    | **Назначение** |
|-----------------|--------------------|----------------|
| 172.16.42.0/32  | 2001:db8:2042:1/64 | R17-R18        |
| 172.16.42.4/32  | 2001:db8:2042:1/64 | R16-R18        |
| 172.16.42.8/32  | 2001:db8:2042:4/64 | R16-R32        |
  
195.178.0.Х/30, 2002:db8 - связь между роутерами между AS
| **IPv4 Subnet** | **IPv6 Subnet**      | **Назначение**              |
|-----------------|----------------------|-----------------------------|
| 195.178.0.0/30  | 2002:db8:1001:101/64 | R14 (AS1001)-R22 (AS101)    |
| 195.178.0.4/30  | 2002:db8:1001:301/64 | R15 (AS1001)-R21 (AS301)    |
| 195.178.0.8/30  | 2002:db8:301:101/64  | R21 (AS301)-R22 (AS101)     |
| 195.178.0.12/30 | 2002:db8:301:520/64  | R21 (AS301)-R24 (AS520)     |
| 195.178.0.16/30 | 2002:db8:101:520/64  | R22 (AS101)-R23 (AS520)     |
| 195.178.0.20/30 | 2002:db8:520:2042/64 | R24 (AS520)-R18(AS2042)     |
| 195.178.0.24/30 | 2002:db8:520:2527/64 | R25 (AS520)-R27(Лабытнанги) |
| 195.178.0.28/30 | 2002:db8:520:2528/64 | R25 (AS520)-R28(Чокурдах)   |
| 195.178.0.32/30 | 2002:db8:520:2628/64 | R26 (AS520)-R28(Чокурдах)   |
| 195.178.0.36/30 | 2002:db8:520:2042/64 | R26 (AS520)-R18(AS2042)     |




192.168.Х.Y/24 / 2003:0:0:X/64   - подсети для конечных пользователей 
| **IPv4 Subnet**  | **IPv6 Subnet** | **Назначение**           |
|------------------|-----------------|--------------------------|
| 192.168.1.0/24   | 2003:0:0:1/64   | VLAN 1 внутри AS1001     |
| 192.168.7.0/24   | 2003:0:0:7/64   | VLAN 7 внутри AS1001     |
| 192.168.8.0/24   | 2003:0:0:8/64   | VLAN 8 внутри AS2042     |
| 192.168.100.0/24 | 2003:0:0:100/64 | VLAN 100 внутри AS2042   |
| 192.168.30.0/24  | 2003:0:0:30/64  | VLAN 30 внутри Чокурдаха |

10.10.1.Х/24 / fc00:777/32  - подсеть для VLAN777- управление 
| **IPv4 Subnet** | **IPv6 Subnet** | **Назначение**            |
|-----------------|-----------------|---------------------------|
| 10.10.0.0/24    | fc00:777/32     | VLAN 777 внутри AS1001    |
| 10.10.1.0/24    | fc00:777/32     | VLAN 777 внутри AS2042    |
| 10.10.2.0/24    | fc00:777/32     | VLAN 777 внутри Чокурдаха |



#### 2. Задокументировать адресное пространство
Следуя принципам из предыдущего раздела, получаем следующие таблицы распределения адресов и VLAN 
#### Москва. AS1001  
##### Таблица адресов и оборудования 

| **Устройстово** | **Интерфейс** | **IPv4 адрес** | **Маска**       | **IPv6 адрес**           | **Описание**    |
|-----------------|---------------|----------------|-----------------|--------------------------|-----------------|
| VPC1            | eth0          | 192.168.1.10   | 255.255.255.0   | 2003:0:0:1::10/64        |                 |
| VPC7            | eth0          | 192.168.7.10   | 255.255.255.0   | 2003:0:0:7::10/64        |                 |
| SW2             | SVI:VLAN777   | 10.10.0.2      | 255.255.255.0   | fc00:777::2/32           |                 |
| SW3             | SVI:VLAN777   | 10.10.0.3      | 255.255.255.0   | fc00:777::3/32           |                 |
| SW4             | SVI:VLAN777   | 10.10.0.4      | 255.255.255.0   | fc00:777::4/32           |                 |
| SW5             | SVI:VLAN777   | 10.10.0.5      | 255.255.255.0   | fc00:777::5/32           |                 |
| R12             | VLAN1         | 192.168.1.12   | 255.255.255.0   | 2003:0:0:1::12/64        |                 |
|                 | VLAN7         | 192.168.7.12   | 255.255.255.0   | 2003:0:0:7::12/64        |                 |
|                 | VLAN777       | 10.10.0.12     | 255.255.255.0   | fc00:777::12/32          |                 |
|                 | e0/2          | 172.16.254.1   | 255.255.255.252 | 2001:db8:1001:1::12/64   | R12-R14         |
|                 | e0/3          | 172.16.254.5   | 255.255.255.252 | 2001:db8:1001:2::12/64   | R12-R15         |
| R13             | VLAN1         | 192.168.1.13   | 255.255.255.0   | 2003:0:0:1::13/64        |                 |
|                 | VLAN7         | 192.168.7.13   | 255.255.255.0   | 2003:0:0:7::13/64        |                 |
|                 | VLAN777       | 10.10.0.13     | 255.255.255.0   | fc00:777::13/32          |                 |
|                 | e0/2          | 172.16.254.9   | 255.255.255.252 | 2001:db8:1001:3::13/64   | R13-R15         |
|                 | e0/3          | 172.16.254.13  | 255.255.255.252 | 2001:db8:1001:4::13/64   | R13-R14         |
| R19             | e0/0          | 172.16.254.17  | 255.255.255.252 | 2001:db8:1001:5::19/64   | R19-R14         |
| R20             | e0/0          | 172.16.254.21  | 255.255.255.252 | 2001:db8:1001:6::20/64   | R20-R15         |
| R14             | e0/0          | 172.16.254.2   | 255.255.255.252 | 2001:db8:1001:1::14/64   | R14-R12         |
|                 | e0/1          | 172.16.254.14  | 255.255.255.252 | 2001:db8:1001:4::14/64   | R14-R13         |
|                 | e0/2          | 195.178.0.1     | 255.255.255.252 | 2002:db8:1001:101::14/64 | R14-R22 (AS101) |
|                 | e0/3          | 172.16.254.18  | 255.255.255.252 | 2001:db8:1001:5::14/64   | R14-R19         |
| R15             | e0/0          | 172.16.254.10  | 255.255.255.252 | 2001:db8:1001:3::15/64   | R15-R13         |
|                 | e0/1          | 172.16.254.6   | 255.255.255.252 | 2001:db8:1001:2::15/64   | R15-R12         |
|                 | e0/2          | 195.178.0.5     | 255.255.255.252 | 2002:db8:1001:301::15/64 | R15-R21 (AS301) |
|                 | e0/3          | 172.16.254.22  | 255.255.255.252 | 2001:db8:1001:6::15/64   | R15-R20         |

##### Таблица VLAN
| **VLAN** | **Название** | **Назначенные интерфейсы**                                              |
|----------|--------------|-------------------------------------------------------------------------|
| 777      | MGMT         | SW3:VLAN777,SW2:VLAN777,SW4:VLAN777,SW5:VLAN777,R12:VLAN777,R13:VLAN777 |
| 1        | VPC1VLAN     | SW3:e0/2, R12:VLAN1,R13:VLAN1                                           |
| 7        | VPC7VLAN     | SW2:e0/2, R12:VLAN7,R13:VLAN7                                           |


#### Ламас. AS301
##### Таблица адресов и оборудования  
| **Устройстово** | **Интерфейс** | **IPv4 адрес** | **Маска**       | **IPv6 адрес**           | **Описание**     |
|-----------------|---------------|----------------|-----------------|--------------------------|------------------|
| R21             | e0/0          | 195.178.0.6     | 255.255.255.252 | 2002:db8:1001:301::21/64 | R21-R15 (AS1001) |
|                 | e0/1          | 195.178.0.9     | 255.255.255.252 | 2002:db8:301:101::21/64  | R21-R22 (AS101)  |
|                 | e0/2          | 195.178.0.13    | 255.255.255.252 | 2002:db8:301:520::21/64  | R21-R24 (AS520)  |

####Киторн. AS101
#####Таблица адресов и оборудования  
| **Устройстово** | **Интерфейс** | **IPv4 адрес** | **Маска**       | **IPv6 адрес**           | **Описание**     |
|-----------------|---------------|----------------|-----------------|--------------------------|------------------|
| R22             | e0/0          | 195.178.0.2     | 255.255.255.252 | 2002:db8:1001:101::22/64 | R22-R14 (AS1001) |
|                 | e0/1          | 195.178.0.10    | 255.255.255.252 | 2002:db8:301:101::22/64  | R22-R21 (AS301)  |
|                 | e0/2          | 195.178.0.17    | 255.255.255.252 | 2002:db8:101:520::22/64  | R22-R23 (AS520)  |

#### Триада AS520
##### Таблица адресов и оборудования
| **Устройстово** | **Интерфейс** | **IPv4 адрес**  | **Маска**       | **IPv6 адрес**           | **Описание**        |
|-----------------|---------------|-----------------|-----------------|--------------------------|---------------------|
| R23             | e0/0          | 195.178.0.18     | 255.255.255.252 | 2002:db8:101:520::23/64  | R23-R22(AS101)      |
|                 | e0/1          | 172.16.52.1/32  | 255.255.255.254 | 2001:db8:520:1::23/64    | R23-R25             |
|                 | e0/2          | 172.16.52.5/32  | 255.255.255.254 | 2001:db8:520:2::23/64    | R23-R24             |
| R24             | e0/0          | 195.178.0.14     | 255.255.255.252 | 2002:db8:301:520::24/64  | R24-R21(AS301)      |
|                 | e0/1          | 172.16.52.9/32  | 255.255.255.254 | 2001:db8:520:3::24/64    | R24-R26             |
|                 | e0/2          | 172.16.52.6/32  | 255.255.255.254 | 2001:db8:520:2::24/64    | R24-R23             |
|                 | e0/3          | 195.178.0.21     | 255.255.255.252 | 2002:db8:520:2042::24/64 | R24-R18(AS2042)     |
| R25             | e0/0          | 172.16.52.2/32  | 255.255.255.254 | 2001:db8:520:1::25/64    | R25-R23             |
|                 | e0/1          | 195.178.0.25     | 255.255.255.252 | 2002:db8:520:2527::25/64 | R25-R27(Лабытнанги) |
|                 | e0/2          | 172.16.52.13/32 | 255.255.255.254 | 2001:db8:520:4::25/64    | R25-R26             |
|                 | e0/3          | 195.178.0.29     | 255.255.255.252 | 2002:db8:520:2528::25/64 | R25-R28(Чокурдах)   |
| R26             | e0/0          | 172.16.52.10/32 | 255.255.255.254 | 2001:db8:520:3::26/64    | R26-R24             |
|                 | e0/1          | 195.178.0.33     | 255.255.255.252 | 2002:db8:520:2628::26/64 | R26-R28(Чокурдах)   |
|                 | e0/2          | 172.16.52.14/32 | 255.255.255.254 | 2001:db8:520:4::26/64    | R26-R25             |
|                 | e0/3          | 195.178.0.37     | 255.255.255.252 | 2002:db8:520:2042::26/64 | R26-R18(AS2042)     |


#### C-Петербург. AS2042 
##### Таблица адресов и оборудования
| Устройстово | Интерфейс   | IPv4 адрес     | Маска           | IPv6 адрес               | Описание       |
|-------------|-------------|----------------|-----------------|--------------------------|----------------|
| VPC8        | eth0        | 192.168.8.10   | 255.255.255.0   | 2003:0:0:8::10/64        |                |
| VPC         | eth0        | 192.168.100.10 | 255.255.255.0   | 2003:0:0:100::10/64      |                |
| SW9         | SVI:VLAN777 | 10.10.1.9      | 255.255.255.0   | fc00:777::9/32           |                |
| SW10        | SVI:VLAN777 | 10.10.1.10     | 255.255.255.0   | fc00:777::10/32          |                |
| R17         | VLAN8       | 192.168.8.17   | 255.255.255.0   | 2003:0:0:8::17/64        |                |
|             | VLAN100     | 192.168.100.17 | 255.255.255.0   | 2003:0:0:100::17/64      |                |
|             | VLAN777     | 10.10.1.17     | 255.255.255.0   | fc00:777::17/32          |                |
|             | e0/1        | 172.16.42.1    | 255.255.255.252 | 2001:db8:2042:1::17/64   | R17-R18        |
| R16         | VLAN8       | 192.168.8.16   | 255.255.255.0   | 2003:0:0:8::16/64        |                |
|             | VLAN100     | 192.168.100.16 | 255.255.255.0   | 2003:0:0:100::16/64      |                |
|             | VLAN777     | 10.10.1.16     | 255.255.255.0   | fc00:777::16/32          |                |
|             | e0/1        | 172.16.42.5    | 255.255.255.252 | 2001:db8:2042:1::16/64   | R16-R18        |
|             | e0/3        | 172.16.42.8    | 255.255.255.252 | 2001:db8:2042:4::16/64   | R16-R32        |
| R32         | e0/0        | 172.16.42.9    | 255.255.255.252 | 2001:db8:2042:4::32/64   | R32-R16        |
| R18         | e0/0        | 172.16.42.6    | 255.255.255.252 | 2001:db8:2042:1::18/64   | R18-R16        |
|             | e0/1        | 172.16.42.2    | 255.255.255.252 | 2001:db8:2042:1::18/64   | R18-R17        |
|             | e0/2        | 195.178.0.22    | 255.255.255.252 | 2002:db8:520:2042::18/64 | R18-R24(AS520) |
|             | e0/3        | 195.178.0.38    | 255.255.255.252 | 2002:db8:520:2042::18/64 | R18-R26(AS520) |

##### Таблица VLAN
| VLAN | Название | Назначенные интерфейсы                              |
|------|----------|-----------------------------------------------------|
| 777  | MGMT     | SW9:VLAN777, SW10:VLAN777, R17:VLAN777, R16:VLAN777 |
| 100  | VPCVLAN  | SW10:e0/2, R17:VLAN100, R16:VLAN100                 |
| 8    | VPC8VLAN | SW9:e0/2, R17:VLAN100, R16:VLAN100                  |


#### Чокурдах
##### Таблица адресов и оборудования
| Устройстово | Интерфейс   | IPv4 адрес    | Маска           | IPv6 адрес               | Описание       |
|-------------|-------------|---------------|-----------------|--------------------------|----------------|
| VPC30       | eth0        | 192.168.30.10 | 255.255.255.0   | 2003:0:0:30::10/64       |                |
| VPC31       | eth0        | 192.168.31.10 | 255.255.255.0   | 2003:0:0:31::10/64       |                |
| SW29        | SVI:VLAN777 | 10.10.2.29    | 255.255.255.0   | fc00:777::29/32          |                |
| R28         | e0/0        | 195.178.0.34   | 255.255.255.252 | 2002:db8:520:2628::28/64 | R28-R26(AS520) |
|             | e0/1        | 195.178.0.30   | 255.255.255.252 | 2002:db8:520:2528::28/64 | R28-R25(AS520) |
|             | e0/2.30     | 192.168.30.1  | 255.255.255.0   | 2003:0:0:30::1/64        |                |
|             | e0/2.31     | 192.168.31.1  | 255.255.255.0   | 2003:0:0:31::1/64        |                |
|             | e0/2.777    | 10.10.2.28    | 255.255.255.0   | fc00:777::28/32          |

##### Таблица VLAN
| VLAN | Название  | Назначенные интерфейсы    |
|------|-----------|---------------------------|
| 777  | MGMT      | SW29:VLAN777,R28:e0/2.777 |
| 30   | VPC30VLAN | SW29:e0/0                 |
| 31   | VPC31VLAN | SW29:e0/1                 |

#### Лабытнанги
| Устройстово | Интерфейс | IPv4 адрес  | Маска           | IPv6 адрес               | Описание       |
|-------------|-----------|-------------|-----------------|--------------------------|----------------|
| R27         | e0/0      | 195.178.0.26 | 255.255.255.252 | 2002:db8:520:2527::27/64 | R27-R25(AS520) |