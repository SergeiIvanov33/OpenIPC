# Установка OpenIPC на IVG-G3S (gk7205v210)
## Установка прошивки
1. Выпаять флеш-память и сделать ее резервную копию с помощью CH341;
2. Очистить флеш-память с помощью CH341;
3. [Создать](<https://openipc.org/supported-hardware/featured>) инструкцию по установке и скачать образ прошивки OpenIPC;
4. Записать данную прошивку во флеш-память камеры с помощью CH341;
5. Впаять флеш-память в камеру;
6. Дальнейшие манипуляции можно производить с помощью CP2102 через UART-порт камеры, ПО-PUTTY, tftpd64.
## Распайка под USB-модем
Чтобы припаяться к нужным контактам на плате, необходимо "сдуть" феном выделенную ниже часть:

![Деталь, которую необходимо "сдуть"](https://github.com/SergeiIvanov33/OpenIPC/blob/master/3.jpg)

Далее необходимо припаять USB-вход к 9(DM) - белый, 10(GND) - черный и 11(DP) - зеленый, расположенным на контактной площадке J3.

![1](https://github.com/SergeiIvanov33/OpenIPC/blob/master/1.jpg)
![2](https://github.com/SergeiIvanov33/OpenIPC/blob/master/2.jpg)

Затем припаять провода на 12В и GND, взятые с контактной площадки, отмеченной красной стрелкой. GND соединить с GND из предыдущего пункта, а 12В перенаправить на вход понижающего преобразователя (12В-5В) (можно использовать разобранный зарядник, работающий от прикуривателя). С выхода преобразователя взять +5В и соединить с красным проводом USB-входа.
## Настройка модема Huawei e3131 (420S (МТС)) для работы с камерой на OpenIPC
**Модем необходимо разблокировать и перепрошить в Hilink.**
Подробная инструкция находится [здесь](<https://www.youtube.com/watch?v=Fh9ysGLFdDM&ab_channel=%D0%90%D0%B2%D0%B8%D1%82%D0%BE%D0%B4%D0%BE%D1%80.%D0%A0%D0%A4>).
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
Затем "передернуть" модем, выполнить команду ```ifup eth1``` или перезагрузить.
Более подробная инструкция [здесь](<https://github.com/OpenIPC/sandbox-fpv/blob/master/lte-fpv.md>).
## Настройка WireGuard-сервера
Для полного удаления WireGuard выполнить следующие команды:\
```sudo apt remove wireguard```\
```sudo apt autoclean & sudo apt autoremove```  

На сервере выполнить следующую команду для того, чтобы скачать последнюю версию скрипта с GitHub:
```curl -O https://raw.githubusercontent.com/angristan/wireguard-install/master/wireguard-install.sh```\
Дать файлу скрипта права на выполнение:\
```chmod +x wireguard-install.sh```\
Посмотреть публичный IP сервера:\
```ip -br a```\
Для запуска скрипта выполнить следующую команду:\
```sudo ./wireguard-install.sh```\
Далее, следуя инструкциям, вводить запрашиваемые данные. Порт, желательно, выбрать 57397.\
Сервер с первым пользователем установлен.\
Запустить WireGuard-службу:
```systemctl start wg-quick@wg0.service```\
Чтобы служба запускалась после перезагрузки сервера:\
```systemctl enable wg-quick@wg0.service```\
Проверить состояние службы:
```systemctl status wg-quick@wg0.service```\
Далее перейти в директорию, где будут хранится конфигурации сервера\
```cd /etc/wireguard/```\
Создать директорию для хранения данных пользователя и перейти в нее:\
```mkdir <name_user> && cd <name_user>```\
Сгенерировать открытый и закрытый ключи для клиента:\
```wg genkey > privatekey```\
```wg pubkey < <name_user>_privatekey > <name_user>_publickey```\
Вывести сгенерированные ключи на экран:\
```tail <name_user>_publickey <name_user>_publickey```\
Отредактировать конфигурационный файл:\
```nano wg0.conf```\
Добавить следующие строки:
```
[Peer]
PublicKey = [<name_user>_publickey]
AllowedIPs = 10.30.0.2/32
```
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
Address = 10.30.0.2/32
DNS = 8.8.8.8

[Peer]
PublicKey = [публичный ключ <name_user>_publickey]
Endpoint =[ ip адрес сервера ]:57397
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 20
```
Затем данные сконфигурированного файла скопировать и внести в новый тоннель пользователя.\
Чтобы удобно добавить VPN на телефон/планшет, на сервере установить программу для генерации qr-кодов:\
```apt install -y qrencode```\
Находясь в каталоге с конфигурацией, выполнить:\
```qrencode -t ansiutf8 -r myphone.conf```.


