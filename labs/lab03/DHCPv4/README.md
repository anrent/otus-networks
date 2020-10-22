# Настройка DHCPv4

### Топология

![Image alt](https://github.com/anrent/otus-networks/blob/main/labs/lab03/DHCPv4/topo.png)

### Таблица адресации


| Device  | Interface          |IP Address    |Subnet Mask    |Default Gateway|
| -------:| ------------------:| ------------:| -------------:| :-------------|
| R1      | F0/1               | 10.0.0.1     |255.255.255.252| N/A           |
|         | F0/0               | N/A          |N/A            | N/A           |
|         | F0/0.100           | 192.168.1.1  |255.255.255.192| N/A           |
|         | F0/0.200           | 192.168.1.65 |255.255.255.224| N/A           |
|         | F0/0.1000          | N/A          |N/A            | N/A           |
| R2      | F0/1               | 10.0.0.2     |255.255.255.252| N/A           |
|         | F0/0               | 192.168.1.97 |255.255.255.240| N/A           |
| S1      | VLAN 200           | 192.168.1.66 |255.255.255.224| 192.168.1.65  |
| S2      | VLAN 1             | 192.168.1.98 |255.255.255.240| 192.168.1.97  |
| PC-A    | NIC                | DHCP         |DHCP           | DHCP          |
| PC-B    | NIC                | DHCP         |DHCP           | DHCP          |



### Таблица VLAN

| VLAN    | Name             |Interface Assigned            |
| :-------|:-----------------| :----------------------------| 
| 1       | N/A              | S2: F0/18                    |
| 100     | Clients          | S1: F0/6                     |
| 200     | Management       | S1: VLAN 200                 |
| 999     | Parking_Lot      | S1: F0/1-4, F0/7-24, G0/1-2  |
| 1000    | Native           | N/A                          |


## Задачи:

#### Часть 1. Создание топологии сети и базовая настройка.
#### Часть 2: Настройка двух DHCP серверов на R1
#### Часть 3: Настройка DHCP Relay на R2




### Часть 1: ССоздание топологии сети и базовая настройка.

#### Cхема лабораторного стенда, выполненная в GNS3:
![Image alt](https://github.com/anrent/otus-networks/blob/main/labs/lab03/DHCPv4/topo.png)

#### Разработка адресации

Нарезаем 192.168.1.0/24 в соответствии со следующими требованиями:

     Одна подсеть, «Подсеть A», поддерживает 58 хостов (клиентская VLAN на R1).

Подсеть A: 192.168.1.0/26 (192.168.1.1–192.168.1.63)

     Одна подсеть, «Подсеть B», поддерживает 28 хостов (управляющая VLAN на R1).

Подсеть B: 192.168.1.64/27 (192.168.1.65–192.168.1.95)

     Одна подсеть, «Подсеть C», поддерживает 12 хостов (клиентская сеть на R2).

Подсеть C: 192.168.1.96/28 (192.168.1.97-192.168.1.111)


#### Команды предварительной настройки

```
enable
conf t
no ip domain-lookup                        //Отключение DNS lookup, чтобы устройство не пыталось преобразовать неправильно введенные команды, как если бы они были именами хостов.
service password-encryption                //Шифрование паролей     
hostname S2                                //R1 R2 S1 соответственно
enable secret class                        //Включение запроса пароля для привелигированного режима
line console 0                             //Конфигурация консольной линии
password cisco                             //Пароль, запршиваемый при подключении к устройству через консоль
logging synchronous                        //Синхронизация консольных лог сообщений с вводом команд
exit
line vty 0 15                              //Конфигурация vty линий
login                                      //Необходимость ввода общего пароля
password cisco                             //Пароль, запршиваемый при подключении к устройству через vty линии
exit
banner motd "Hello there"                  //Баннер
interface vlan 1
ip address 192.168.1.1 255.255.255.0       //Адреса в таблице адресации
exit
copy running-config startup-config         //сохранение конфигурации
```

#### Настройки маршрутизации между VLAN на  R1

```
R1(config)# interface f0/0.100                              // создание подинтерфейса для vlan 100
R1(config-subif)# description Client Network                // описание интерфейса
R1(config-subif)# encapsulation dot1q 100                   // включение инкапсуляции dot1q для создания trunk канала
R1(config-subif)# ip address 192.168.1.1 255.255.255.192    // настройка ip адреса для интерфейса, используя первый адрес подсети

R1(config-subif)# interface f0/0.200                        // создание подинтерфейса для vlan 200
R1(config-subif)# description Management Network            // описание интерфейса
R1(config-subif)# encapsulation dot1q 200                   // включение инкапсуляции dot1q для создания trunk канала
R1(config-subif)# ip address 192.168.1.65 255.255.255.224   // настройка ip адреса для интерфейса, используя первый адрес подсети

R1(config-subif)# interface f0/0.1000                       // создание подинтерфейса для vlan 200
R1(config-subif)# description Native VLAN                   // описание интерфейса
R1(config-subif)# encapsulation dot1q 1000 native           // включение инкапсуляции dot1q для native vlan
```

#### Настройка F0/0 на R2, затем F0/1 и настройка статических маршрутов на обеих роутерах

```
R2(config)# interface f0/0
R2(config-if)# ip address 192.168.1.97 255.255.255.240      // настройка ip адреса для интерфейса, используя первый адрес подсети
R2(config-if)# no shutdown
R2(config-if)# exit

R1(config)# interface f0/1
R1(config-if)# ip address 10.0.0.1 255.255.255.252          // настройка ip адреса для интерфейса в соответствии с таблицей адресации
R1(config-if)# no shutdown
R1(config)# ip route 0.0.0.0 0.0.0.0 10.0.0.2               // настройка маршрута по умолчанию до R2

R2(config)# interface f0/1
R2(config-if)# ip address 10.0.0.2 255.255.255.252          // настройка ip адреса для интерфейса в соответствии с таблицей адресации
R2(config-if)# no shutdown
R2(config)# ip route 0.0.0.0 0.0.0.0 10.0.0.1               // настройка маршрута по умолчанию до R1
```

#### Создание VLAN на коммутаторе S1

Создание vlan согласно таблице:

```
S1(config)# vlan 100
S1(config-vlan)# name Clients
S1(config-vlan)# vlan 200
S1(config-vlan)# name Management
S1(config-vlan)# vlan 999
S1(config-vlan)# name Parking_Lot
S1(config-vlan)# vlan 1000
S1(config-vlan)# name Native
S1(config-vlan)# exit
```

Настройка vlan 200 в качесте интерфейса управления на коммутаторе S1:

```
S1(config)# interface vlan 200
S1(config-if)# ip address 192.168.1.66 255.255.255.224
S1(config-if)# no shutdown
S1(config-if)# exit
S1(config)# ip default-gateway 192.168.1.65                 // настройка шлюза по умолчанию, его роль выполняет R1
```
Настройка vlan 1 в качесте интерфейса управления на коммутаторе S2:

```
S2(config)# interface vlan 1
S2(config-if)# ip address 192.168.1.98 255.255.255.240
S2(config-if)# no shutdown
S2(config-if)# exit
S2(config)# ip default-gateway 192.168.1.97                 // настройка шлюза по умолчанию, его роль выполняет R2
```


Отключение неиспользуемых портов на S1 и S2:

```
S1(config)# interface range e0/1–2
S1(config-if-range)# switchport mode access
S1(config-if-range)# switchport access vlan 999
S1(config-if-range)# shutdown
S1(config-if-range)# exit
```

```
S2(config)# interface range e0/1 – 2
S2(config-if-range)# switchport mode access
S2(config-if-range)# shutdown
S2(config-if-range)# exit
```

#### Настройка VLAN на интерфейсах коммутаторов

Настройка пользовательского интерфейса на коммутаторе S1:

```
S1(config)# interface e0/3
S1(config-if)# switchport mode access        // в режиме доступа
S1(config-if)# switchport access vlan 100    // пользовательский VLAN
```

Настройка магистрального интерфейса на коммутаторе S1:

```
S1(config)# interface e0/0
S1(config-if)# switchport mode trunk                             //в режиме транка
S1(config-if-range)# switchport trunk native vlan 1000           //native vlan для магистрального интерфейса
S1(config-if-range)# switchport trunk allowed vlan 100,200,1000  //vlan разрешённые для работы в транке
```

Согласно заданию, VLAN на S2 не настраивается - доступ происходит через default vlan 1.

### Часть 2: Настройка и проверка DHCP на маршрутизаторе R1

#### Настройка DHCP пулов на две подсети на R1

Настройка пула для сети 192.168.1.0/24:

```
R1(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.5    //исключаем из пула раздачи первые 5 адресов
R1(config)# ip dhcp pool R1_Client_LAN                          //создаём первый клиентский пул
R1(dhcp–config)# network 192.168.1.0 255.255.255.192            //присваем пулу диапазон адресов
R1(dhcp–config)# domain-name ccna-lab.com                       //настройка доменного имени
R1(dhcp–config)# default-router 192.168.1.1                     //шлюз по умолчанию для пула
R1(dhcp–config)# lease 2 12 30                                  //время жизни выданных адресов 2 дня 12 часов 30 минут
```

Настройка пула для сети 192.168.1.96/28:

```
R1(config)# ip dhcp excluded-address 192.168.1.97 192.168.1.101
R1(config)# ip dhcp pool R2_Client_LAN
R1(dhcp–config)# network 192.168.1.96 255.255.255.240
R1(dhcp–config)# default-router 192.168.1.97
R1(dhcp–config)# domain-name ccna-lab.com
R1(dhcp–config)# lease 2 12 30
```

### Часть 3: Настройка и проверка DHCP Relay на маршрутизаторе R2

```
R2(config)# interface f0/0                   // интерфейс, с которого будут ретранслироваться входящие запросы на R1
R2(config-if)# ip helper-address 10.0.0.1    // адрес DHCP сервера - R1
R2(config-if)# exit
```

#### Проверка настройки DHCP

Вывод комманды show ip dhcp pool:

![Image alt](https://github.com/anrent/otus-networks/blob/main/labs/lab03/DHCPv4/show_ip_dhcp_pool.png)

Вывод комманды show ip dhcp bindings:

![Image alt](https://github.com/anrent/otus-networks/blob/main/labs/lab03/DHCPv4/ip_dhcp_binding.png)

Вывод комманды show ip dhcp server statistics:

![Image alt](https://github.com/anrent/otus-networks/blob/main/labs/lab03/DHCPv4/show_ip_dhcp_pool.png)


Как видно по скриншотам, настройка верна, все клиенты получают ip адреса.

Проверяем доступность между конечными клиентами:

![Image alt](https://github.com/anrent/otus-networks/blob/main/labs/lab03/DHCPv4/ping.png)

Клиенты видят друг-друга, настройка оборудования верна.
