---
layout: post
title:  "Запускаем GUI приложения в LXD, еще и со звуком"
date:   2021-06-25 22:11:33 +0300
categories: jekyll github-pages
---
### Собственно сабж

За основу поста, взят этот
[пост](https://blog.simos.info/how-to-run-graphics-accelerated-gui-apps-in-lxd-containers-on-your-ubuntu-desktop/).
Но GUI, у меня, сразу не взлетел, как и звук. Пришлось гуглить. Как оказалось,
для Arch (Manjaro), нужные немного другие настройки Х-ов. Звук в итоге тоже
заработал. Подробности ниже.

### Подготовка

Для начала, созданим и настроим, новый контейнер.
```sh
$ lxc launch ubuntu:x guiapps
$ lxs shell
```

`lxc shell` это алиас из [перд.
поста](https://dvg4000.github.io/some-notes/jekyll/github-pages/2021/04/19/lxd-lxc.html) `lxc alias add shell "exec @ARGS@ -- su -l
ubuntu"`. Он подключается к выбранному контейнеру и логинится под юзером ubuntu.
Это пользователь по умолчанию в Ubuntu Linux. *(Заметка) Надо будет переделать этот alias чтобы можно было передавать имя пользователя.*

Внутри конейнера, устанавливаем след. пакеты.
```sh
ubuntu@guiapps:~$ sudo apt update
ubuntu@guiapps:~$ sudo apt install x11-apps
ubuntu@guiapps:~$ sudo apt install mesa-utils
ubuntu@guiapps:~$ sudo apt install alsa-utils
ubuntu@guiapps:~$ sudo apt install pulseaudio
ubuntu@guiapps:~$ exit
```

В оригинальном [посте](https://blog.simos.info/how-to-run-graphics-accelerated-gui-apps-in-lxd-containers-on-your-ubuntu-desktop/), не было устновки `pulseaudio`. Но без этого пакета, у меня
не работал звук. Про `pulseaduio` нашел в комментах к посту. 

Далее нужно спроецировать ID пользователя хоста (текущего пользователя) в
контейнер. Делается это, как показано ниже. Более подробная информация, есть в
оригинальном
[посте](https://blog.simos.info/how-to-run-graphics-accelerated-gui-apps-in-lxd-containers-on-your-ubuntu-desktop/).

```sh
$ echo "root:$UID:1" | sudo tee -a /etc/subuid /etc/subgid
$ lxc config set guiapps raw.idmap "both $UID 1000"
$ lxc restart guiapps
```

### Настройка графики

Пробрасываем Unix-сокет X сервера, в контейнер.
```sh
$ lxc config device add guiapps X0 disk path=/tmp/.X11-unix/X0 source=/tmp/.X11-unix/X0
```

Далее нужно пробросить в контейнер, файл авторизации для подключения к Х
серверу (может это и не так называется). В оригинальном посте, это делается
след. командой.
```sh
$ lxc config device add guiapps Xauthority disk path=/home/ubuntu/.Xauthority source=${XAUTHORITY}
```

Но у меня это не заработало - контейнер не мог подключиться к X серверу, хоста.
Ругался на протокол. На помощь пришел гугол:
[reddit](https://www.reddit.com/r/archlinux/comments/c7b9yw/gui_apps_inside_of_lxd_containers/esrvb4y?utm_source=share&utm_medium=web2x&context=3),
[stackoverflow](https://stackoverflow.com/questions/16296753/can-you-run-gui-applications-in-a-linux-docker-container/25280523#25280523),
[arch.wiki](https://wiki.archlinux.org/title/Systemd-nspawn#Avoiding_xhost).
В итоге мне помогло следущее.
```sh
$ XAUTH=/tmp/guiapps_xauth
$ xauth nextract - "$DISPLAY" | sed -e 's/^..../ffff/' | xauth -f "$XAUTH" nmerge -
$ lxc config device add guiapps Xauthority disk path=/home/ubuntu/.Xauthority source=${XAUTH}
```

Далее включаем аппаратное ускорение графики в контейнере.
```sh
$ lxc config device add guiapps mygpu gpu
$ lxc config device set guiapps mygpu uid 1000
$ lxc config device set guiapps mygpu gid 1000
```

Задаем экран и пробуем.
```sh
$ echo $DISPLAY
:0
$ lxc shell
ubuntu@guiapps:~$ echo "export DISPLAY=:0" >> ~/.profile 
ubuntu@guiapps:~$ source .profile
ubuntu@guiapps:~$ xclock
```
Должны появиться часики.<br/>
![xclock](https://i2.wp.com/blog.simos.info/wp-content/uploads/2017/05/xclock.png?ssl=1)

`glxgears` у меня тоже заработали. Показывали 60 fps.<br/>
![glxgears](https://i1.wp.com/blog.simos.info/wp-content/uploads/2017/05/glxgears-container.png?ssl=1)

### Звук

Для работы звука, на хосте нужно разрешить доступ по сети, к локальным устройствам.
Делается это, через установку `paprefs`, далее вкладка `Network Server / [x] Enable network 
access to local sound devices`.
```sh
$ sudo apt install paprefs
$ paprefs
```

![paprefs](https://i2.wp.com/blog.simos.info/wp-content/uploads/2017/05/paprefs-network-access.png?resize=643%2C322&ssl=1)

Далее нужно в контейнере указать адрес сервера Pulseaduio и пробросить куки для
него.
```sh
$ lxc shell
ubuntu@guiapps:~$ echo export PULSE_SERVER="tcp:`ip route show 0/0 | awk '{print $3}'`" >> ~/.profile
ubuntu@guiapps:~$ mkdir -p ~/.config/pulse/
ubuntu@guiapps:~$ echo export PULSE_COOKIE=/home/ubuntu/.config/pulse/cookie >> ~/.profile
ubuntu@guiapps:~$ exit
$ lxc config device add guiapps PACookie disk path=/home/ubuntu/.config/pulse/cookie source=/home/${USER}/.config/pulse/cookie
```

Пробуем звук.
```sh
$ lxc shell
ubuntu@pulseaudio:~$ speaker-test -Dpulse -c6 -twav
```

P.S. Если что-то не заработало, можно попробовать перезапустить контейнер.
```sh
$ lxc restart guiapps
```
