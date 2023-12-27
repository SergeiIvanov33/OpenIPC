# Установка OpenIPC на IVG-G3S (gk7205v210)
## Установка прошивки
1. Выпаять флеш-память и сделать ее резервную копию с помощью CH341;
2. Очистить флеш-память с помощью CH341;
3. [Создать](<https://openipc.org/supported-hardware/featured>) инструкцию по установке и скачать образ прошивки OpenIPC;
4. Записать данную прошивку во флеш-память камеры с помощью CH341;
5. Впаять флеш-память в камеру;
6. Дальнейшие манипуляции можно производить с помощью CP2102 через UART-порт камеры, ПО-PUTTY, tftpd64.
---
## Распайка под USB-модем
Чтобы припаяться к нужным контактам на плате, необходимо "сдуть" феном выделенную ниже часть:

![Деталь, которую необходимо "сдуть"](https://github.com/SergeiIvanov33/OpenIPC/blob/master/3.jpg)

Далее необходимо припаять USB-вход к 9(DM) - белый, 10(GND) - черный и 11(DP) - зеленый, расположенным на контактной площадке J3.

![1](https://github.com/SergeiIvanov33/OpenIPC/blob/master/1.jpg)
![2](https://github.com/SergeiIvanov33/OpenIPC/blob/master/2.jpg)

Затем припаять провода на 12В и GND, взятые с контактной площадки, отмеченной красной стрелкой. GND соединить с GND из предыдущего пункта, а 12В перенаправить на вход понижающего преобразователя (12В-5В) (можно использовать разобранный зарядник, работающий от прикуривателя). С выхода преобразователя взять +5В и соединить с красным проводом USB-входа.

---
## Настройка модема Huawei e3131 (420S (МТС)) для работы с камерой на OpenIPC
**Модем необходимо разблокировать и перепрошить в Hilink.**
Подробная инструкция находится [здесь](<https://www.youtube.com/watch?v=Fh9ysGLFdDM&ab_channel=%D0%90%D0%B2%D0%B8%D1%82%D0%BE%D0%B4%D0%BE%D1%80.%D0%A0%D0%A4>).

---
## Интеграция модема с камерой
На камере установить usb_modeswitch, выполнив следующие команды:
```
curl -o /usr/sbin/usb_modeswitch http://fpv.openipc.net/files/usb-modeswitch/musl/usb_modeswitch && chmod +x /usr/sbin/usb_modeswitch
curl -o /usr/lib/libusb-1.0.so.0.3.0 http://fpv.openipc.net/files/usb-modeswitch/musl/libusb-1.0.so.0.3.0 && chmod +x /usr/lib/libusb-1.0.so.0.3.0
ln -s -f /usr/lib/libusb-1.0.so.0.3.0 /usr/lib/libusb-1.0.so
ln -s -f /usr/lib/libusb-1.0.so.0.3.0 /usr/lib/libusb-1.0.so.0
```
Альтернативное хранилище:
```
curl -o /usr/sbin/usb_modeswitch http://fpv.openipc.net/files/usb-modeswitch/glibc/usb_modeswitch && chmod +x /usr/sbin/usb_modeswitch
curl -o /usr/lib/libusb-1.0.so.0.3.0 http://fpv.openipc.net/files/usb-modeswitch/glibc/libusb-1.0.so.0.3.0 && chmod +x /usr/lib/libusb-1.0.so.0.3.0
ln -s -f /usr/lib/libusb-1.0.so.0.3.0 /usr/lib/libusb-1.0.so
ln -s -f /usr/lib/libusb-1.0.so.0.3.0 /usr/lib/libusb-1.0.so.0
ln -s -f /lib/libc-2.32.so /lib/libc.so
```
Затем внести этот текст в файл /etc/network/interfaces.d/eth1 (создать файл, если отсутствует):
```
auto eth1
iface eth1 inet dhcp
    pre-up sleep 4
    pre-up if [ ! -z "`lsusb | grep 12d1:1f01`" ]; then usb_modeswitch -v 0x12d1 -p 0x1f01 -J; fi
    pre-up if [ ! -z "`lsusb | grep 12d1:14dc`" ]; then modprobe usbserial vendor=0x12d1 product=0x14dc; fi
    pre-up modprobe rndis_host
    pre-up sleep 2
```
Затем "передернуть" модем, выполнить команду ```ifup eth1``` или перезагрузить.\
Более подробная инструкция [здесь](<https://github.com/OpenIPC/sandbox-fpv/blob/master/lte-fpv.md>).

---
## Установка и настройка WireGuard-сервера
Для полного удаления WireGuard выполнить следующие команды:\
```sudo apt remove wireguard```\
```sudo apt autoclean & sudo apt autoremove```  

На сервере выполнить следующую команду для того, чтобы скачать последнюю версию скрипта с GitHub:\
```curl -O https://raw.githubusercontent.com/angristan/wireguard-install/master/wireguard-install.sh```\
Дать файлу скрипта права на выполнение:\
```chmod +x wireguard-install.sh```\
Посмотреть публичный IP сервера:\
```ip -br a```\
Для запуска скрипта выполнить следующую команду:\
```sudo ./wireguard-install.sh```\
Далее, следуя инструкциям, вводить запрашиваемые данные. Порт, желательно, выбрать 57397.\
Сервер с первым пользователем установлен.\
Запустить WireGuard-службу:\
```systemctl start wg-quick@wg0.service```\
Чтобы служба запускалась после перезагрузки сервера:\
```systemctl enable wg-quick@wg0.service```\
Проверить состояние службы:\
```systemctl status wg-quick@wg0.service```\

---
## Добавление клиентов WireGuard
Перейти в директорию, где будут хранится конфигурации клиентов:\
```cd /etc/wireguard/```\
Создать директорию для хранения данных пользователя и перейти в нее:\
```mkdir <name_user> && cd <name_user>```\
Сгенерировать открытый и закрытый ключи для клиента:\
```wg genkey > <name_user>_privatekey```\
```wg pubkey < <name_user>_privatekey > <name_user>_publickey```\
Вывести сгенерированные ключи на экран:\
```tail <name_user>_publickey <name_user>_privatekey```\
Отредактировать конфигурационный файл:\
```nano /etc/wireguard/wg0.conf```\
Добавить следующие строки:
```
[Peer]
PublicKey = [<name_user>_publickey]
AllowedIPs = *.*.*.*/32
```
Следует ввести ip-адрес клиента в сети VPN в поле ```Allowed IPs```.\
Сохранить файл и перезапустить службу:\
```systemctl restart wg-quick@wg0```\
Проверить статус службы:\
```systemctl status wg-quick@wg0```\
Создать конфигурационный файл для пользователя:\
```touch <name_user>.conf```\
Открыть его:\
```nano <name_user>.conf```\
Внести следующие строки:
```
[Interface]
PrivateKey = [приватный ключ <name_user>_privatekey]
Address = <ip, введенный в wg0 для <name_user>/32
DNS = 8.8.8.8

[Peer]
PublicKey = [публичный ключ сервера]*
Endpoint = [ip адрес сервера]:57397
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 20
```
***[публичный ключ сервера] можно посмотреть, введя в терминале сервера команду ```wg```**.\
Затем данные сконфигурированного файла скопировать и внести в новый тоннель пользователя.\
Чтобы удобно добавить VPN на телефон/планшет, на сервере установить программу для генерации qr-кодов:\
```apt install -y qrencode```\
Находясь в каталоге с конфигурацией, выполнить:\
```qrencode -t ansiutf8 -r <name_user>.conf```\
Более подробную информацию по установке сервера можно найти [здесь](https://losst.pro/prostaya-nastrojka-wireguard-linux), а по настройке сервера и клиента - [здесь](<https://profitserver.ru/knowledge-base/nastroyka-wireguard-vpn-na-svoem-servere>). 

---
## Настройка WireGuard на IVG-G3S
1. Открыть файл ```wireguard.conf``` на камере:\
```vi /etc/wireguard.conf```\
Внести в него следующие строки:
```
[Peer]
AllowedIPs = 10.66.66.0/24
Endpoint = 95.66.153.204:57397
PersistentKeepalive = 25
PublicKey = kb/f5vmkVzJodrYTlDbA616++OFjitGHVADjV+3sXUw= 
```
- AllowedIPs - подсеть VPN;
- Endpoint - ip-сервера и используемый порт;
- PublicKey - публичный ключ сервера.
2. Открыть файл ```wg0``` на камере:\
```/etc/network/interfaces.d/wg0```\
Внести в него следующие строки:
```
auto wg0
iface wg0 inet static
    address 10.66.66.6
    netmask 255.255.255.0
    pre-up modprobe wireguard
    pre-up ip link add dev wg0 type wireguard
    pre-up wg setconf wg0 /etc/wireguard.conf
    post-down ip link del dev wg0
```
Здесь нужно заменить только ```address``` - ip-адрес камеры в сети VPN.





