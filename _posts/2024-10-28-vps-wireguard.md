---
layout: post
title:  "VPS + WireGuard + Dante"
date:   2024-10-26 14:30:33 +0300
tags: [VPS, WireGuard, Proxy, Dante]
---

В этом посте, будет описание настройки **VPS**, с **VPN WireGuard** и прокси сервером **Dante**.

Погнали.

## Обозначения
`$` - выполнением комадны, под учеткой пользователя.
<br>`#` - выполнением комадны, под root, на локальном хосте (домашнем компуктере например).
<br>`$` - выполнением комадны, под учеткой пользователя, на локальном хосте (домашнем компуктере например).
<br>`$(vps)` - выполнением комадны, под учеткой пользователя, на **VPS**.

## Настройка VPS

Выбираем VPS провайдера. Я остановился на [Timeweb Cloud](https://timeweb.cloud/). 
<br>Конфигурация VPS: `Proc 1*3.3ГГц / RAM 1Гб / NVMe 15Гб / Net 200 Мбит`.
<br>ОСь: `Ubuntu 22.04`.

### Настраиваем рабочую учетку на VPS

Через веб-консоль, на сайте провайдера VPS, логинимся в VPS под root. У [Timeweb Cloud](https://timeweb.cloud/) там все удобной сделано.

Добавляем учетку `user123` и сразу добавляем эту учетку в группу `sudo`. 
```sh
#(vps) useradd -m user123
#(vps) user123
#(vps) usermod -aG sudo user123
```

### Настройки безопасности VPS

#### SSH

На локальном компе, генерим `ssh-ключ` и сразу копируем его на VPS.
```sh
$ ssh-keygen -t ed25519 -C "VPS" -f ed25519_vps 
$ ssh-copy-id -i ~/.ssh/ed25519_vps.pub user123@<vps-ip>
$ ssh -i .ssh/ed25519_vps user123@<vps-ip> 
```

`<vps-ip>` - это публиный IP4-адресс, VPS.

Далее, включаем доступ только по `ssh-ключам`, меняем стандартный порт и разрешаем доступ только нашей учетке, на VPS.
Для редактирования файлов, я использую `vim`. Если не знакомы с `vim`, то лучше попробуйте `nano`.
```sh
#(vps) vim /etc/ssh/sshd_config
```
Оставляем раскомментированными только указанные ниже строчки.
```conf
Port 2128
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
X11Forwarding no
AllowUsers user123
```

Перезапускаем ssh-сервис.
```sh
#(vps) systemctl restart sshd.service
```

После этих настроек, можно логинится по `ssh` на `VPS` и закрывать Web-консоль.
```sh
$ ssh -P 2128 -i path/to/generated/ed25519_vps user123@<vps-ip>
```

Можно вообще [запретить](https://www.tecmint.com/disable-root-login-in-linux/) root учетку. Но это по желанию. Я не стал заморачиваться.

#### Обновляем ОСь и меняем shell

Обновляемся.
```sh
$(vps) sudo apt update
$(vps) sudo apt upgrade
```

Меняем `sh` на `bash`. `chsh` запросит указать новый шел, надо ввести `/bin/bash`.
```sh
$(vps) chsh 
```

#### Защита SSH от подбора пароля. Fail2ban

У **Timeweb Cloud**, есть [статья по настройке Fail2Ban](https://timeweb.cloud/docs/unix-guides/block-ssh-brute-force-attacks-fail2ban). 
Уже не помню, но кажется я ей пользовался при настройке `Fail2Ban` на `VPS`.

Устанавливаем Fail2Ban.
```bash
$(vps) sudo apt install fail2ban
```

Создаем фалй настроек, со следующим содержимым.
```
$(vps) sudo nano /etc/fail2ban/jail.local
```

```conf
[sshd]  
enabled  = true  
port = 2128
findtime = 120  
maxretry = 3  
bantime = 43200
```

Стартуем сервис `fail2ban`:
```bash
$(vps) sudo systemctl restart fail2ban.service
```

Смотрим статус:
```bash
$(vps) sudo fail2ban-client status sshd
```

Если вдруг, сами попали в бан, то заходим в web-консоль, смотрим статус (как показано выше), находим свой ip и убираем его из бана.
```
$(vps) sudo fail2ban-client unban <ip>
```

#### Настройка фаервола. UFW

[Статья](https://timeweb.cloud/tutorials/ubuntu/nastrojka-faervola-v-ubuntu-s-pomoschyu-utility-ufw) на **Timeweb Cloud**. 
Еще одна [статья](https://www.cyberciti.biz/faq/how-to-set-up-ufw-firewall-on-ubuntu-24-04-lts-in-5-minutes/).

**UFW** должен быть установлен на **Ubuntu** из коробки.

Отключаем добавление IPv6 правил.
```bash
sudo nano /etc/default/ufw
```
Строчку `IPV6=yes` меняем на `IPV6=no`

Добавляем правила.
```bash
$(vps) sudo ufw default deny incoming
$(vps) sudo ufw default allow outgoing
$(vps) sudo ufw allow 2128
$(vps) sudo ufw enable
$(vps) sudo ufw status verbose
$(vps) sudo systemctl status ufw.service
```

#### WireGuard

За основу настройки, взяли вот эти две статьи: [digitalocean.com](https://www.digitalocean.com/community/tutorials/how-to-set-up-wireguard-on-ubuntu-22-04), 
[shibumi.dev](https://shibumi.dev/posts/isolated-clients-with-wireguard/).

##### Настройка серверной чаcти (VPS)

Устанавливаем `WireGuard`.
```bash
$(vps) sudo apt install wireguard
```

Генерим ключи.
```bash
$(vps) wg genkey | sudo tee /etc/wireguard/private.key
$(vps) sudo chmod go= /etc/wireguard/private.key
$(vps) sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key
```


Находим внешний интерфейс, через который сервер выходит в интернет. В моем случае, это `eth0`. 
```bash
$(vps) ip route list default
```
`default via <vps-public-ip> dev eth0 proto dhcp src <vps-public-ip> metric 100`

Пишем конифг виртуального-сетевого-интерфеса (wg0).
```bash
$(vps) sudo vim /etc/wireguard/wg0.conf
```
```conf
[Interface]
# base64_encoded_server_private_key - это содержимое /etc/wireguard/private.key
PrivateKey = base64_encoded_server_private_key
Address = 192.168.99.1/24
ListenPort = 51616
SaveConfig = true

# Client 1
#[Peer]
#PublicKey = base64_encoded_client_public_key 
#AllowedIPs = 192.168.99.2/32

# Client 2
#[Peer]
#PublicKey = base64_encoded_client_public_key 
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
$(vps) sudo vim /etc/sysctl.conf
```

Раскомментируем строчку.
```txt
net.ipv4.ip_forward=1
```

Применяем изменения.
```bash
$(vps) sudo sysctl -p
```

Открываем в фаерволе (UFW), порт для WireGuard.
```bash
$(vps) sudo ufw allow 51616/udp
```

Включаем и запускаем, Wireguard.
```bash
$(vps) sudo systemctl enable wg-quick@wg0.service
$(vps) sudo systemctl start wg-quick@wg0.service
```

##### Настройка клиентской части (Linux)

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
PrivateKey = base64_encoded_client_private_key
Address = 192.168.99.2

[Peer]
# base64_encoded_server_public_key - это содержимое /etc/wireguard/public.key на **VPS**.
PublicKey = base64_encoded_server_public_key
AllowedIPs = 192.168.99.0/24
Endpoint =  217.151.230.55:51616
```

**Перенаправление всего трафика.**
<br>Если в конфиге, `AllowedIPs` указать следующим образом, `AllowedIPs = 192.168.99.0/24`, тогда WireGuard будет
перенаправлять весь трафик с машины, через VPS. 

Прежде чем включать WireGuard на клиенте, стоит добавить **публичный** ключ клиента, на сервере (VPS).
<br>Сделать это можно с коммандной строки на VPS.
```bash
$(vps) sudo wg set wg0 peer base64_encoded_client_public_key allowed-ips 192.168.99.2
```

Либо через редактирование конфига `/etc/wireguard/wg0.conf` на VPS. Раскомментируем строки под # Client 1:
```conf
# Client 1
[Peer]
PublicKey = base64_encoded_client_public_key 
AllowedIPs = 192.168.99.2/32
```

Перезапускаем WireGuard на VPS.
```bash
$(vps) sudo systemctl restart wg-quick@wg0.service
```

Далее можно включать WireGuard на клиенте.
```bash
$ sudo wg-quick@wg0 up wg0
```

Выключить можно так:
```bash
$ sudo wg-quick@wg0 down wg0
```

##### Настройка клиентской части (Windows)

Генерим ключи. 
<Br>Ключи можно сгенерить на винде, но можно и на linux-машине. Что я и сделал. Там же сформировал `wg0.conf`:
```conf
[Interface]
# base64_encoded_client_private_key - это содержимое private.key
PrivateKey = base64_encoded_client_private_key
Address = 192.168.99.3

[Peer]
# base64_encoded_server_public_key - это содержимое /etc/wireguard/public.key на **VPS**.
PublicKey = base64_encoded_server_public_key
AllowedIPs = 192.168.99.0/24
Endpoint =  217.151.230.55:51616
```

1. Качаем [windows-клиента Wireguard](https://download.wireguard.com/windows-client/). 
2. Логинимся в учетную запись Администратора, либо в учетную запись с админскими провами. Запуск с правами администратор из-под учетки пользователя, не поможет.
3. Загружаем wg0.conf в клиенте и подключаемся там же.

Для удобного включения и отключения vpn под обычной учеткой, достаточно выполнить следующую команду, с правами администратора. 
```cmd
netsh interface set interface wg0 (enable | disable)` 
```

##### Настройка клиентской части (Android)

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
