# Настройка DHCPv4

### Топология

![Image alt](https://github.com/anrent/otus-networks/blob/main/labs/lab03/DHCPv6/topo.png)

### Таблица адресации

| Device  | Interface        | IPv6 address                 |
| :-------|:-----------------| :----------------------------| 
| R1      | F0/0             | 2001:db8:acad:2::1 /64       |
|         |                  | Sfe80::1                     |
|         | F0/1             | 2001:db8:acad:1::1/64        |
|         |                  | fe80::1                      |
| R2      | F0/0             | 2001:db8:acad:2::2 /64       |
|         |                  | Sfe80::2                     |
|         | F0/1             | 2001:db8:acad:3::1/64        |
|         |                  | fe80::1                      |
| PC-A    | NIC              | DHCP                         |
| PC-B    | NIC              | DHCP                         |

### Задачи
#### Часть 1: Создание и базовая настройка сети
#### Часть 2: Проверка получения адреса через SLAAC от маршрутизатора R1
#### Часть 3: Настройка и проверка Stateless DHCPv6 на маршрутизаторе R1
#### Часть 4: Настройка и проверка Stateful DHCPv6 на маршрутизаторе R1
#### Часть 5: Настройка и проверка DHCPv6 Relay на маршрутизаторе R2


### Часть 1: Создание и базовая настройка сети

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
ipv6 unicast-routing                       //Настройка только для R1 и R2, позволяющая включить IPv6 маршрутизацию
banner motd "Hello there"                  //Баннер
exit
copy running-config startup-config         //сохранение конфигурации
```
#### Настройка интерфейсов и маршрутов на R1 и R2
В соответствии с таблицей адресации настраиваем интерфейсы

Для маршрутизатора R1:
```
interface f0/0
ipv6 address fe80::1 link-local
ipv6 address 2001:db8:acad:2::1/64
no shutdown
interface f0/1
ipv6 address fe80::1 link-local
ipv6 address 2001:db8:acad:1::1/64
no shutdown
ipv6 route ::/0 2001:db8:acad:2::2      //Маршрут по умолчанию в сторону R2
```
Для маршрутизатора R2:
```
interface f0/0
ipv6 address fe80::2 link-local
ipv6 address 2001:db8:acad:2::1/64
no shutdown
interface f0/1
ipv6 address fe80::1 link-local
ipv6 address 2001:db8:acad:3::2/64
no shutdown
ipv6 route ::/0 2001:db8:acad:2::1      //Маршрут по умолчанию в сторону R1
```

Проверяем правильность настройки, R2 пингуется с R1:

![Image alt](https://github.com/anrent/otus-networks/blob/main/labs/lab03/DHCPv6/ping_R2.png)


### Часть 2: Проверка получения адреса через SLAAC от маршрутизатора R1

При включении PC1 автоматически получил IPv6 адрес в сети 2001:db8:1::/64, при этом оба маршрутизатора доступны с компьютера.

![Image alt](https://github.com/anrent/otus-networks/blob/main/labs/lab03/DHCPv6/SLAAC_PC1.png)

