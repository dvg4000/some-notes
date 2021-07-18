---
layout: post
title:  "VMware Player + SVN = snapshots"
date:   2021-07-18 20:55:33 +0300
tags: [VMware, SVN, PCEm, 86Box, Windows XP, Windows 98, VirtualBox, Sandboxie, DOXBox]
---
### Сабж
Собственно соль данного эссе в том, что мне уже давно хочется поиграть в старые
игры. Ностальгия понимаете ли. Но не все старые игры, можно купить легально. А
устанавливать, бесплатно купленные игры, на основную систему, как-то боязно. 
При этом, некоторые старые игры, могут не запуститься на новой ОСи / железе.
К счастию, интернет, полон ~~всякого ...~~ всяких полезных программ, которые
(не)легко могут помочь с сабжем.

Что мне известно и про что будет в посте.
- [DOSBox](https://www.dosbox.com/),[DOXBox
  Stagin](https://dosbox-staging.github.io/), [DOSBox-X](https://dosbox-x.com/) - отлично подходят для DOS-игр. 
  А на последнем, даже можно запустить Windows 98 и даже с 3D-ускорением (сам не
  пробовал). DOSBox отличная вещь, рекомендую для DOS-игр. Есть
  еще [ScummVM](https://www.scummvm.org/), но это больше для квестов. Так же
  рекомендую, заглянуть
  [сюда](https://blog.archive.org/2019/10/13/2500-more-ms-dos-games-playable-at-the-archive/).
  Здесь можно поиграть прямо в ~~браузере~~ интернет обозревателе.
- [PCEm](http://pcem-emulator.co.uk/) и его клон [86Box](https://github.com/86Box/86Box) - эмуляторы старого железа. 
Как пишет [Old-Games.ru Wiki](https://www.old-games.ru/wiki/PCem), PCem — это эмулятор персонального компьютера на уровне регистров аппаратного обеспечения. 
Из-за этого (а может и нет), чтобы сэмулировать Pentium 233 MMX, понадобится хороший современный процессор. 
С другой стороны, из-за этого (а может и нет), устанавливаемая ОС, всает как на
реальное железо. Планирую попробовать его с Windows 98. Будет отдельный пост
- [Wine](https://www.winehq.org/) - must have на Linux. Умеет старые и новые
  windows-игры. Конечно не все, но много. Благодаря [Valve / Steam](http://store.steampowered.com/) и их
  [Proton](https://www.protondb.com/), игр работающих в Linux, становится все больше. Есть удобные утилиты. 
  Сам пользуюсь вот этой - [Lutris](https://lutris.net/). На Linux это все можно
  изолировать, всяческий. Например
  [так](https://dvg4000.github.io/some-notes/jekyll/github-pages/2021/06/25/lxd-gui-audio.html)
  или при помощи [firejail](https://wiki.archlinux.org/title/firejail) и наверно
  кучей других способов. Но посто сегодня не про него. :)
- [VirtualBox](https://www.virtualbox.org/) - годнота. Но начиная с 6.1,
  отключили поддержку 3D в Windows XP
  [1](https://forums.virtualbox.org/viewtopic.php?f=s&t=98113),
  [2](https://forums.virtualbox.org/viewtopic.php?f=2&t=96434). А мне хочется
  именно Windows XP.
- [VMware Player](https://www.vmware.com/ru/products/workstation-player.html) -
  умеет как 3D, причем вплоть до Direct X11, так и Windows XP. :) Внимательный
  читатель спросить - а причем тут собственно SVN. И будет прав. )) Cоль в
  том, что VMware Player, не умеет делать снимки (snapshots) виртуальных машин.
  Делать их умеет VMware Workstation, которая стоит денег, в отличии от Player.
  А почему именно SVN, а не Git, об этом уже в посте.

### Пост
Так почему SVN? Потому что я нашел статью - [снимки для VMware
Player](http://unix.ba/text/multiple-snapshots-in-vmware-player/). И в этой
статье, про SVN. :) Я конечно же больше привык к Git, и началь искать про Git и
большие двоичные файлы. Но то что нашел, не перевесило чашу весов в сторону Git.
  
#### Настройка SVN
Я немного отошел от статья, но не сильно. Все делал на `Windows 10`. Использовал
[TortoiseSVN](https://tortoisesvn.net/). В качестве вирутально ОСи, использовал `Windows XP`. 
Вирутальные машины я держу в каталоге `D:\VirtualPC`. 

1. Создаем каталог `D:\VirtualPC\WindowsXP`. Вунтри создаем два подкаталога:
   `repo` и  `work`.
2. Внутри каталога `repo` создаем SVN-репозиторий. На винде, это делается через вызов
   контесктного меню каталога, `TortoiseSVN / Create`. В посте это делается через команду
   `svnadmin create <path_to_repo>`.
3. Внутри каталога `work` создаем рабочую копию. Так же вызываем конектсное
   меню, выбираем пункт `TortoiseSVN / Checkout`. В появившемся диалоге, указываем путь до
   `D:\VirtualPC\WindowsXP\repo`. В посте это делается командой
   `svn checkout file:///<path_to_repo>`

#### Установка Windows XP
Здесь все просто - берете и ставите. :) Только в качестве каталога виртуальной машины, нужно выбрать каталог `work`, в моем случае это был `D:\VirtualPC\WindowsXP\work`.

После завершения установки, рекмоендую сразу поставить VMware Tools. Если пункт
меню `Player / Manage / Install VMware Tools` заблокирован (серый), то попробуйте сделать как написано в этом [посту](https://www.ghacks.net/2019/11/08/how-to-install-vmware-tools-if-the-option-is-grayed-out/
), а именно:
- выключить виртуальную машину (ВМ)
- зайти в настрйоки ВМ
- удалить устройство CD-ROM
- добавить устройство CD-ROM
- в настройках CD-ROM выбрать `physical autodetect`
Далее в меню `Player / Manage` выбираем `Install VMware Tools` и следуем мастеру устанвки.

![windows xp](https://i.imgur.com/tMx1pon.png)

#### Снимок
Теперь у нас есть чистая Windows XP, внутри виртуальной машины. Самое время
сделать снимок (snapshot).

Добавляем отслеживаемые файлы: 
```sh
svn add *.nvram, *.vmdk, *.vmsd, *.vmx, *.vmxf
```
На самом деле, я не выполнял этой команды, все это я сделал через конеткстное
меню. В оригинальном посте, автора добавляет все файлы в рабочем каталоге. 
Я не стал добавлять файлы логов и катлог кэша. Их я добавил в ignore-лист.
```sh
svn propset svn:ignore "*.log" also_cache_dir
```
Эту команду я тоже не выполнял, а всопользовался контекстным меню.

Коммитим (commit). В посте это делается командой 
```sh
svn commit -m 'Initial commit'
```

Теперь, для восстановления на определенный снимок, достаточно выбрать нужную
ревизию в логах, и восстановить (update) рабочий каталог `work` на эту ревизию.
```sh
$ svn log
$ svn update -r2
```
Либо воспользоваться контекстным меню.

### P.S. 

Полезные утилиты.<br>
[WinCDEmu](https://wincdemu.sysprogs.org/) - бесплатный CD/DVD/BD эмулятор, с
открытым исходным кодом. Работает на Windows XP - 10. Поддерживает много форматов.
