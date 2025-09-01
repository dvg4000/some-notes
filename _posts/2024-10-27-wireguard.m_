---
layout: post
title:  "WireGuard"
date:   2024-10-27 12:46:33 +0300
tags: [WireGuard, VPS, Ubuntu]
---

В этом посте, будет описание настройки [**WireGuard**](https://www.wireguard.com/) на **VPS** c **Ubuntu 22.04**.

Предполагается, что на настраиваемой системе, уже настроен доступ по SSH, а так же на VPS используется UFW, в качестве фаервола.

## Обозначения
`$` - выполнением комадны, под учеткой пользователя.
<br>`#` - выполнением комадны, под root, на локальном хосте (домашнем компуктере например).
<br>`$` - выполнением комадны, под учеткой пользователя, на локальном хосте (домашнем компуктере например).
<br>`(vps)$` - выполнением комадны, под учеткой пользователя, на **VPS**.

## База

За основу настройки, взяли вот эти две статьи: [digitalocean.com](https://www.digitalocean.com/community/tutorials/how-to-set-up-wireguard-on-ubuntu-22-04), 
[shibumi.dev](https://shibumi.dev/posts/isolated-clients-with-wireguard/).

## Настройка серверной чаcти (VPS)

Устанавливаем `WireGuard`.
```bash
(vps)$ sudo apt install wireguard
```

Генерим ключи.
```bash
(vps)$ wg genkey | sudo tee /etc/wireguard/private.key
(vps)$ sudo chmod go= /etc/wireguard/private.key
(vps)$ sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key
```

Находим внешний интерфейс, через который сервер выходит в интернет. В моем случае, это `eth0`. 
```bash
(vps)$ ip route list default
```
`default via <vps-public-ip> dev eth0 proto dhcp src <vps-public-ip> metric 100`

Пишем конифг виртуального-сетевого-интерфеса (wg0).
```bash
(vps)$ sudo vim /etc/wireguard/wg0.conf
```
```conf
[Interface]
# base64_encoded_server_private_key - это содержимое /etc/wireguard/private.key
PrivateKey = <base64_encoded_server_private_key>
Address = 192.168.99.1/24
ListenPort = 51616
SaveConfig = true

# Client 1
#[Peer]
#PublicKey = <base64_encoded_client_public_key>
#AllowedIPs = 192.168.99.2/32

# Client 2
#[Peer]
#PublicKey = <base64_encoded_client_public_key>
#AllowedIPs = 192.168.99.3/32

PostUp = ufw route allow in on wg0 out on eth0
PostUp = iptables -t nat -I POSTROUTING -o eth0 -j MASQUERADE
PostUp = iptables -I FORWARD -i wg0 -o wg0 -j REJECT --reject-with icmp-net-prohibited
PreDown = ufw route delete allow in on wg0 out on eth0
PreDown = iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
PreDown = iptables -D FORWARD -i wg0 -o wg0 -j REJECT --reject-with icmp-net-prohibited
```

Разрешаем проброс (между интерфесами) в ОСи.
```bash
(vps)$ sudo vim /etc/sysctl.conf
```

Раскомментируем строчку.
```txt
net.ipv4.ip_forward=1
```

Применяем изменения.
```bash
(vps)$ sudo sysctl -p
```

Открываем в фаерволе (UFW), порт для WireGuard.
```bash
(vps)$ sudo ufw allow 51616/udp
```

Включаем и запускаем, Wireguard.
```bash
(vps)$ sudo systemctl enable wg-quick@wg0.service
(vps)$ sudo systemctl start wg-quick@wg0.service
```

## Настройка клиентской части (Linux)

Генерим ключи.
```bash
$ wg genkey | sudo tee /etc/wireguard/private.key
$ sudo chmod go= /etc/wireguard/private.key
$ sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key
```

Пишем конифг виртуального-сетевого-интерфеса (wg0).
```bash
$ sudo vim /etc/wireguard/wg0.conf
```
```conf
[Interface]
# base64_encoded_client_private_key - это содержимое /etc/wireguard/private.key
PrivateKey = <base64_encoded_client_private_key>
Address = 192.168.99.2

[Peer]
# base64_encoded_server_public_key - это содержимое /etc/wireguard/public.key на **VPS**.
PublicKey = <base64_encoded_server_public_key>
AllowedIPs = 192.168.99.0/24
Endpoint =  <VPS-IP>:51616
```

**Перенаправление всего трафика.**
<br>Если в конфиге, `AllowedIPs` указать следующим образом, `AllowedIPs = 192.168.99.0/24`, тогда WireGuard будет
перенаправлять весь трафик с машины, через VPS. 

Прежде чем включать WireGuard на клиенте, стоит добавить **публичный** ключ клиента, на сервере (VPS).
<br>Сделать это можно с коммандной строки на VPS.
```bash
(vps)$ sudo wg set wg0 peer base64_encoded_client_public_key allowed-ips 192.168.99.2
```

Либо через редактирование конфига `/etc/wireguard/wg0.conf` на VPS. Раскомментируем строки под # Client 1:
```conf
# Client 1
[Peer]
PublicKey = <base64_encoded_client_public_key>
AllowedIPs = 192.168.99.2/32
```

Перечитываем конфиги WireGuard на VPS.
```bash
(vps)$ sudo systemctl restart wg-quick@wg0.service
```

Далее можно включать WireGuard на клиенте.
```bash
$ sudo wg-quick@wg0 up wg0
```

Выключить можно так:
```bash
$ sudo wg-quick@wg0 down wg0
```

Полезности для Linux:

Создаем шаблон для конфигов `wg.conf.template`.
```conf
[Interface]
PrivateKey = the_key
Address = 192.168.99.100
[Peer]
PublicKey = <base64_encoded_server_public_key>
AllowedIPs = 192.168.99.0/24
Endpoint = <VPS IP>
```

Создаем новый конфиг.
```sh
wg genkey | xargs -i sed -r '2s#(PrivateKey = ).+#\1{}#' wg0.conf.template > wg0.conf
```

Получаем публичный ключ из конфига, который потом добавляем на сервер.
```sh
grep -oP '^PrivateKey\s=\s\K(.+)$' wg0.conf | wg pubkey
```

Генерим QR-код
```sh
qrencode -t png -r wg0.conf -o ~/Desktop/qr.png
```

## Настройка клиентской части (Windows)

Генерим ключи. 
<Br>Ключи можно сгенерить на винде, но можно и на linux-машине. Что я и сделал. Там же сформировал `wg0.conf`:
```conf
[Interface]
# base64_encoded_client_private_key - это содержимое private.key
PrivateKey = <base64_encoded_client_private_key>
Address = 192.168.99.3

[Peer]
# base64_encoded_server_public_key - это содержимое /etc/wireguard/public.key на **VPS**.
PublicKey = <base64_encoded_server_public_key>
AllowedIPs = 192.168.99.0/24
Endpoint =  <VPS-IP>:51616
```

1. Качаем [windows-клиента Wireguard](https://download.wireguard.com/windows-client/). 
2. Логинимся в учетную запись Администратора, либо в учетную запись с админскими провами. Запуск с правами администратор из-под учетки пользователя, не поможет.
3. Загружаем wg0.conf в клиенте и подключаемся там же.

Для удобного включения и отключения vpn под обычной учеткой, достаточно выполнить следующую команду, с правами администратора. 
```cmd
netsh interface set interface wg0 (enable | disable)` 
```

<!-- Android client 
## Настройка клиентской части (Android)

Качаем [клиента](https://www.wireguard.com/install/) с официального сайта WireGuard или из [Google Play](https://play.google.com/store/apps/details?id=com.wireguard.android).

Генерим ключи. 
<br>Ключи и конфиг проще создать на linux-машине.

<br>Далее используя `qrencode`, можно сгенерить qr-code из конфига:
пямо в консоль:
```bash
$ qrencode -t ansiutf8 wg-client.conf
```

или в файл:
```bash
$ qrencode -t png -o client-qr.png -r wg-client.conf
```

Далее сканируем, полученный qr-код из android-клиента и готово. В настройках клиента, можно указать, какие приложения будут использовать WireGuard.
-->
