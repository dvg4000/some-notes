---
layout: post
title:  "Android Studio Emulator networks"
date:   2022-02-09 20:23:33 +0300
tags: [Android, WebView]
---
### Если нужно с эмулятора, подключиться к порту хоста
Хост в данном случае - это компьютер на которм запущен эмулятор. 
Если кратко, то адрес хоста такой - `10.0.2.2`. Т.е. если на хосте запустить
виртуальный http-сервер.
```sh
python3 -m http.server 10123
```
А в эмулторе открыть Chrome и перейти по адресу `http://10.0.2.2:10123`, то
Chrome покажет то что отдает запущенный http-сервер (содержимое каталога, в
котором запущен сервер).
Более подробно,
[здесь](https://developer.android.com/studio/run/emulator-networking).

### Редирект порта на хосте, на порт в эмуляторе
[Здесь](https://developer.android.com/studio/run/emulator-networking) же, но
ниже. Делает это через команду `redir add <protocol>:<host-port>:<guest-port>`.
Где `<guest-port>` - это порт в эмуляторе.
Пример.
```sh
telnet localhost 5554
```
При подключении, будет предложение выполнить команду `auth <auth_token>`. Где
находится `<auth_token>`, будет написано там же. Если команду `auth` не выполнить,
то команда `redir` будет недоступна.
Далее.
```sh
redir add tcp:5000:6000
```
Теперь все что прилетает на 5000 порт хоста (127.0.0.1:5000), должно
редиректиться на 6000 порт эмулятора (10.0.2.15:6000).

### Android WebView и самопальный SSL-сертификат
Понадобилось из Android-приложения, в WebView, зайти по https на хост. При
попытке зайти на `https://10.0.2.2:10123`, WebView показывает белый экран.
Гугол и SO подсказали следующий вариант
[решения](https://stackoverflow.com/questions/7416096/android-webview-not-loading-an-https-url)
задачи.
```java
@Override
public void onReceivedSslError(WebView view, SslErrorHandler handler, SslError error) {
        handler.proceed(); // Ignore SSL certificate errors
}
```
В продовой версии, этого делать не стоит.
