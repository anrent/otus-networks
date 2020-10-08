# VLAN и маршрутизация между VLAN
    
## Исходные данные:

#### Addressing Table

| Device  | Interface          |IP Address   |Subnet Mask  |Default Gateway|
| -------:|------------------:| -------------:| -----------:| :-------------|
| R1      | G0/0/1.3           | 192.168.3.1  |255.255.255.0| N/A           |
|         | G0/0/1.4           | 192.168.4.1  |255.255.255.0| N/A           |
|         | G0/0/1.8           | N/A          |N/A          | N/A           |
| S1      | VLAN 3             | 192.168.3.11 |255.255.255.0| 192.168.3.1   |
| S2      | VLAN 3             | 192.168.3.12 |255.255.255.0| 192.168.3.1   |
| PC-A    | NIC                | 192.168.3.3  |255.255.255.0| 192.168.3.1   |
| PC-B    | NIC                | 192.168.4.3  |255.255.255.0| 192.168.4.1   |

#### VLAN Table

| VLAN    | Name             |Interface Assigned   
| :-------|:-----------------| :-----------| 
| 3       | Management       | S1: VLAN 3  |
|         |                  | S2: VLAN 3  |
|         |                  | S1: F0/6    |
| 4       | Operations       | S2: F0/18   |
| 7       | ParkingLot       | S1: F0/2-4, F0/7-24, G0/1-2   |
|         |                  | S2: F0/2-17, F0/19-24, G0/1-2 |
| 8       | Native           | N/A  |

## Задачи:

#### Часть 1. Создание топологии сети и базовая настройка.
#### Часть 2: Создание VLAN
#### Часть 3: Настройка 802.1Q между коммутаторами
#### Часть 4: Настройка маршрутизации между VLAN на маршрутизаторе
#### Часть 5: Проверка работоспособности


## Выполнение задания:

### Часть 1: Создание сети и настройка основных параметров устройства.

#### Cхема лабораторного стенда, выполненная в Packet Tracer:
![Image alt](https://github.com/anrent/otus-networks/blob/main/labs/lab01/topo.PNG)

Команды предварительной настройки
```
enable
conf t
no ip domain-lookup                        //Отключение DNS lookup, чтобы устройство не пыталось преобразовать неправильно введенные команды, как если бы они были именами хостов.
service password-encryption                //Шифрование паролей     
hostname S2                                //R1 S1 соответственно
ip domain-name S2.local                    //R1.local S1.local соответственно
username admin privilege 15 secret admin   //Создание локальной учётной записи
enable secret class                        //Включение запроса пароля для привелигированного режима
crypto key generate rsa 1024               //Генерация RSA ключа
ip ssh version 2                           //Включение протокола SSH2 для безопасного удалённого доступа
line console 0                             //Конфигурация консольной линии
password cisco                             //Пароль, запршиваемый при подключении к устройству через консоль
logging synchronous                        //Синхронизация консольных лог сообщений с вводом команд
exit
line vty 0 15                              //Конфигурация vty линий
transport input ssh                        //Разрешённый протокол для удалённого доступа SSH2
login                                      //Необходимость ввода общего пароля
password cisco                             //Пароль, запршиваемый при подключении к устройству через vty линии
exit
banner motd "Hello there"                  //Баннер

Дополнительно для R1:
clock set 21:45:00 7 october 2020          //Установка времени
conf t
ntp master 1                               //Настройка R1 в качестве ntp сервера
exit
copy running-config startup-config         //сохранение конфигурации

Дополнительно для S1 и S2:
conf t
ntp server 192.168.3.1                     //Указание в качестве источника времени ntp сервер R1
exit
copy running-config startup-config         //сохранение конфигурации
```
### Часть 2: Создание VLAN

- Создание VLAN на S1 и S2
```
(Команды выполнены в конфигурационном режиме)
vlan 3                                     //создание vlan
name Management                            //назначение имени 
exit
vlan 4                                  
name Operation                           
exit
vlan 7
name ParkingLot
exit
vlan 8
name Native
exit
```
- Конфигурация интерфейса управления и шлюза по умолчанию
```
interface Vlan3
 description Management
 ip address 192.168.3.11 255.255.255.0   //интерфейс управления S1, для S2 192.168.3.12
!
ip default-gateway 192.168.3.1           //шлюз по умолчанию
``` 
- Назначения VLAN 

|VLAN| Name |Ports|
| :-------|:-----------------| -----------:| 
|3|    Management |  S1: Fa0/1(Access) Gi0/1-2(Trunk) S2: Gi0/1(Trunk)|
|4|    Operations |  S1: Gi0/0-1(Trunk) S2: Fa0/1(Access) Gi0/1(Trunk)|
|7|    ParkingLot |  S1: Fa0/2-24(Access) S2: Fa0/2-24, Gig0/2(Access)|

### Часть 3: Настройка 802.1Q между коммутаторами
Следующая настройка справедлива для S1(Gi0/1 Gi0/2) и S2(Gi0/1)
```
interface gigabitEthernet 0/2            // интерфейс в сторону соседнего коммутатора
 switchport mode trunk                   // обозначение интерфейса в виде trunk
 switchport trunk native vlan 8          // назначение Vlan 8 не тегированным 
 switchport trunk allowed vlan 3,4,8     // добавление vlan в trunk
```


### Часть 4: Настройка маршрутизации между VLAN на маршрутизаторе
Настройка интерфейса GigabitEthernet0/0/0 на маршрутизаторе R1 в сторону коммутатора S1
```
interface GigabitEthernet0/0/0
 no ip address
 duplex auto
 speed auto
!
interface GigabitEthernet0/0/0.3
 encapsulation dot1Q 3
 ip address 192.168.3.1 255.255.255.0
!
interface GigabitEthernet0/0/0.4
 encapsulation dot1Q 4
 ip address 192.168.4.1 255.255.255.0
!
interface GigabitEthernet0/0/0.8
 no ip address
!
```
### Часть 5: Проверка работоспособности

- ping с PC-A на шлюз и адреса из сети 192.168.4.0/24: 

![Image alt](https://github.com/anrent/otus-networks/blob/main/labs/lab01/A.PNG).

- ping с PC-B на S1 и S2:

![Image alt](https://github.com/anrent/otus-networks/blob/main/labs/lab01/B.PNG).

### -[Конфигурация сетевого оборудования](config/).
