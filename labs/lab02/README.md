# Развертывание коммутируемой сети с резервными каналами

## Задачи
#### Часть 1. Создание сети и настройка основных параметров устройства
#### Часть 2. Выбор корневого моста
#### Часть 3. Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов
#### Часть 4. Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов


## Часть 1:	Создание сети и настройка основных параметров устройства

Топология сети, построенная в Packet Tracer:

![Image alt](https://github.com/anrent/otus-networks/blob/main/labs/lab02/topo.PNG)

Команды предварительной настройки
```
enable
conf t
no ip domain-lookup                        //Отключение DNS lookup, чтобы устройство не пыталось преобразовать неправильно введенные команды, как если бы они были именами хостов.
service password-encryption                //Шифрование паролей     
hostname S2                                //R1 S1 соответственно
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
ip address 192.168.1.1 255.255.255.0       //Адреса для S2 - .1 дял S3 - .3
exit
copy running-config startup-config         //сохранение конфигурации
```
Проверка работоспособности:

Доступность S2 от S1:

![Image alt](https://github.com/anrent/otus-networks/blob/main/labs/lab02/s1-s2.PNG)

Доступность S3 от S1:

![Image alt](https://github.com/anrent/otus-networks/blob/main/labs/lab02/s1-s3.PNG)

Доступность S3 от S2:

![Image alt](https://github.com/anrent/otus-networks/blob/main/labs/lab02/s2-s3.PNG)
