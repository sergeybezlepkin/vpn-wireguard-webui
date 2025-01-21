[English](/README_ENG.md)

# Запускаем VPN-сервер WireGuard c web-интерфейсом

[WireGuard можно узнать здесь ](https://goo.su/xKp8kj)

Для запуска нужен VPS/VDS сервер. Сервер можно найти [тут](https://goo.su/IsDeUO), или [тут - меньше чем за 3$ в месяц, или на неделю -  меньше чем за 1$](https://vdska.ru/#vds_calc). Операционная система -  Ubuntu 20 или новее. Для запуска VPN-сервера в контейнере достаточно 1 ядра, 1 гигабайта оперативной памяти и от 15 гигабайт свободного пространства. После покупки сервера вам будут выданы учетные данные для входа на сервер. Как войти в терминал на сервере, найдете [тут](https://goo.su/BBH9F).

# Оглавление

- [Ubuntu](#Ubuntu)
  - [Устанавливаем git, curl, nano и Docker](#Устанавливаем-git,-curl,-nano-и-Docker)
  - [Редактируем файл compose.yml для запуска VPN-сервера](##Редактируем-файл-compose.yml-для-запуска-VPN-сервера)
  - [Генерируем хэш для пароля](#Генерируем-хэш-для-пароля)
  - [Узнаем свой маршрутизируемый IP-адрес](#Узнаем-свой-маршрутизируемый-IP-адрес)
  - [Итоговый файл compose.yml](#Итоговый-файл-compose.yml)
  - [(Опционально) Редактируем переменные в файле compose.yml](#(Опционально)-Редактируем-переменные-в-файле-compose.yml)
  - [Добавляем правила в брандмауэр iptables и включаем IP Forward](#Добавляем-правила-в-брандмауэр-iptables-и-включаем-IP-Forward)
  - [Запускаем сервер](#Запускаем-сервер)
  - [Запускаем веб-интерфейс](#Запускаем-веб-интерфейс)
- [(Опционально) Запускаем мониторинг Beszel с оповещением о состоянии в Telegram](#(Опционально)-Запускаем-мониторинг-Beszel-с-оповещением-о-состоянии-в-Telegram)
  - [Настройка уведомлений через Telegram](#Настройка-уведомлений-через-Telegram)
  - [(Опционально) Создаем бота и получаем API токен](#(Опционально)-Создаем-бота-и-получаем-API-токен)


# Ubuntu

## Устанавливаем git, curl, nano и Docker

Устанавливаем git

```sh
sudo apt install -y git
```
Клонируем репозиторий
```sh
git clone https://github.com/sergeybezlepkin/vpn-wireguard.git
```
Переходим в каталог vpn-wireguard
```sh
cd vpn-wireguard/
```
Добавляем права на выполнения файла install_soft и запускаем
```sh
chmod +x install_soft
```
```sh
./install_soft
```

При установке и настройки пакета `iptables-persistent` вам нужно будет ответить на вопрос: `Save current IPv4 rules? [yes/no]` отвечаем `yes`, и на второй вопрос: `Save current IPv6 rules? [yes/no]` отвечаем `yes`.

> !После установки cервер будет перезапущен!

## Редактируем файл compose.yml для запуска VPN-сервера
```sh
cd vpn-wireguard/
```
```sh
nano compose.yml
```

`Файл compose.yml`

```sh
version: "3.7"
services:
  wg-easy:
    image: ghcr.io/wg-easy/wg-easy
    container_name: wg-easy
    restart: unless-stopped
    volumes:
      - etc_wireguard:/etc/wireguard
    environment:
      - LANG=ru
      - WG_HOST=
      - PASSWORD_HASH=
      - UI_TRAFFIC_STATS=true
      - UI_CHART_TYPE=3
      - WG_PORT=49888
      - PORT=48999
      - WG_DEFAULT_ADDRESS=10.45.0.x
      - WG_DEFAULT_DNS=1.1.1.1, 8.8.8.8
      - WG_ALLOWED_IPS=0.0.0.0/0, ::/0
      - WG_PERSISTENT_KEEPALIVE=25
      # - WG_PRE_UP=echo "Pre Up" > /etc/wireguard/pre-up.txt
      # - WG_POST_UP=echo "Post Up" > /etc/wireguard/post-up.txt
      # - WG_PRE_DOWN=echo "Pre Down" > /etc/wireguard/pre-down.txt
      # - WG_POST_DOWN=echo "Post Down" > /etc/wireguard/post-down.txt
      - WG_ENABLE_ONE_TIME_LINKS=true
      - UI_ENABLE_SORT_CLIENTS=true
      - WG_ENABLE_EXPIRES_TIME=true
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    network_mode: host
volumes:
  etc_wireguard:
```

`Оставляем окно терминала открытым. Его нужно будет дополнить своими данными. Открываем вторую вкладку в терминале.`

## Генерируем хэш для пароля

```sh
cd vpn-wireguard/
```
```sh
cat hash
```
```sh
docker run --rm -it ghcr.io/wg-easy/wg-easy wgpw 'YOUR_PASSWORD'
```
После загрузки и запуска образа, терминал выдаст HASH-строку записываем 
> PASSWORD_HASH='$2a$12$o1s1n4ruKZ/2RbnVOUoHOutLY6raO4IDZv7/5z/Qh.K0UbA8RyaVG'

Копируем полученную строку и вставляем её в переменную окружения `PASSWORD_HASH=` в файле `compose.yml`. Добавить её нужно после знака равенства и без кавычек. В hash-строке есть символы $, добавляем по одному дополнительному символу $ рядом с каждым существующим. 

Получаем итог: `PASSWORD_HASH=$$2a$$12$$o1s1n4ruKZ/2RbnVOUoHOutLY6raO4IDZv7/5z/Qh.K0UbA8RyaVG`

## Узнаем свой маршрутизируемый IP-адрес

```sh
curl ifconfig.me
```
`или`
```sh
curl ipinfo.io
```

Полученную строку с адресом добавляем в переменную окружения `WG_HOST=` после знака равенства и без кавычек.

Получаем: `- WG_HOST=178.125.85.124` 

#### Итоговый файл compose.yml

```
version: "3.7"
services:
  wg-easy:
    image: ghcr.io/wg-easy/wg-easy
    container_name: wg-easy
    restart: unless-stopped
    volumes:
      - etc_wireguard:/etc/wireguard
    environment:
      - LANG=ru
      - WG_HOST=178.125.85.124
      - PASSWORD_HASH=$$2a$$12$$o1s1n4ruKZ/2RbnVOUoHOutLY6raO4IDZv7/5z/Qh.K0UbA8RyaVG
      - UI_TRAFFIC_STATS=true
      - UI_CHART_TYPE=3
      - WG_PORT=49888
      - PORT=48999
      - WG_DEFAULT_ADDRESS=10.45.0.x
      - WG_DEFAULT_DNS=1.1.1.1, 8.8.8.8
      - WG_ALLOWED_IPS=0.0.0.0/0, ::/0
      - WG_PERSISTENT_KEEPALIVE=25
      # - WG_PRE_UP=echo "Pre Up" > /etc/wireguard/pre-up.txt
      # - WG_POST_UP=echo "Post Up" > /etc/wireguard/post-up.txt
      # - WG_PRE_DOWN=echo "Pre Down" > /etc/wireguard/pre-down.txt
      # - WG_POST_DOWN=echo "Post Down" > /etc/wireguard/post-down.txt
      - WG_ENABLE_ONE_TIME_LINKS=true
      - UI_ENABLE_SORT_CLIENTS=true
      - WG_ENABLE_EXPIRES_TIME=true
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    network_mode: host
volumes:
  etc_wireguard:
```

`Сохраняем итоговый файл compose.yml, нажав клавиши Ctrl + O. Вводим имя файла — compose.yml — и подтверждаем сохранение` 

## (Опционально) Редактируем переменные в файле compose.yml

- В переменной окружения `LANG=ru` можно изменить язык интерфейса. Доступные варианты: en, ua, ru, tr, no, pl, fr, de, ca, es, ko, vi, nl, is, pt, chs, cht, it, th, hi, ja, si.
- В переменной окружения `PORT=48999` можно указать удобный для себя порт. Например, выберите свободный порт из диапазонов 48658—48999 или 49001—49150.
- В переменной окружения `WG_DEFAULT_ADDRESS=10.45.0.x` можно изменить вторую октету адреса на 8 или 6. Например, получится 10.8.0.x или 10.6.0.x.
- После внесения нужных данных сохраните файл compose.yml.


## Добавляем правила в брандмауэр iptables и включаем IP Forward.

```sh
cat net.ipv4.ip_forward = 1 >> /etc/sysctl.conf
```
```sh
sudo sysctl -p
```

Для правильного добавления правил в iptables необходимо указать имя своего сетевого интерфейса.

Выводим список интерфейсов, состояние, адрес:

```sh
ip -br a
```
Получаем:
| `Интерфейс`     | Статус | IPv4/IPv6 адрес                 |
|---------------|--------|----------------------------------|
| lo            | UNKNOWN| 127.0.0.1/8 ::1/128               |
| `enp0s3`        | UP     | 192.168.1.12/24 fe80::a00:27ff:fe90:fe8d/64 |
| docker0       | DOWN   | 172.17.0.1/16 fe80::42:abff:febb:86ce/64 |
| docker_gwbridge | UP   | 172.20.0.1/16 fe80::42:31ff:fe46:73b3/64 |
| veth4550803@if8 | UP   | fe80::a404:7ff:fed5:585e/64      |
| veth8d014ec@if22 | UP | fe80::1ca2:29ff:feee:9a00/64      |

Теперь добавляем команды в терминал: 
```sh
sudo iptables -t nat -I POSTROUTING 1 -s 10.8.0.0/24 -o YOUR_INTERFACE -j MASQUERADE
```
где, `10.8.0.0/24` - свой IP адрес из переменной окружения `- WG_DEFAULT_ADDRESS=10.8.0.x` и `eth0` имя своего интерфейса.
Пример:
```sh
sudo iptables -t nat -I POSTROUTING 1 -s 10.45.0.0/24 -o enp0s3 -j MASQUERADE
```
Далее, нужно подставить свой интерфейс и применить каждую комманду:

```sh
sudo iptables -I INPUT 1 -i wg0 -j ACCEPT
```
```sh
sudo iptables -I FORWARD 1 -i YOUR_INTERFACE -o wg0 -j ACCEPT
```
```sh
sudo iptables -I FORWARD 1 -i wg0 -o YOUR_INTERFACE -j ACCEPT
```
```sh
sudo iptables -I INPUT 1 -i YOUR_INTERFACE -p udp --dport 49888 -j ACCEPT
```

> Установленный пакет iptables-persistent позволяет сохранить правила iptables после перезагрузки сервера.

## Запускаем сервер

```sh
cd vpn-wireguard/
```
```sh
docker compose up -d
```
`Команда ниже покажет запущенные контейнеры. Ищем колонку STATUS, где должно отображаться состояние UP (healthy)`

```sh
docker ps -a | grep wg-easy
```

## Запускаем веб-интерфейс

Переходим в браузер и вводим IP-адрес узла из переменной `WG_HOST=XXX.XXX.XXX`, добавляем двоеточие и номер порта из переменной `PORT=XXXX`. Пример: http://178.125.85.124:48999.
Попадаем на главную страницу с полем ввода пароля. Вводим пароль, который был создан с использованием hash-строки. После успешного входа откроется интерфейс. Нажимаем кнопку "+ Создать" или "+ Создать клиента", вводим имя клиента. Конфигурация будет готова. 

У каждого клиента есть:

- Тумблер для включения/выключения.
- Кнопка QR-кода для сканирования.
- Кнопка загрузки конфигурации.
- Кнопка удаления клиента.

### [Загрузить клиент под свои устройства](https://www.wireguard.com/install/)

С помощью пакета nmap проверяем доступность открытого порта (указанного в переменной WG_PORT) из интернета.

```sh
sudo nmap -sU -p 49888 $(curl ifconfig.me)
```

Будет запись 

```
PORT      STATE         SERVICE
49888/udp open|filtered unknown

Nmap done: 1 IP address (1 host up) scanned in 2.40 seconds
```

Состояние OPEN указывает на то, что порт доступен и принимает соединения.

### (Опционально) Запускаем мониторинг Beszel с оповещением о состоянии в Telegram

[Beszel можно узнать здесь ](https://beszel.dev/) 

```sh
cd beszel
```
Запускаем: 

```sh
docker compose up -d
```

Комманда ниже, покажет запущенный контейнер, ищем колонку `STATUS` где и увидим состояние `UP (healthy)`

```sh
docker ps -a | grep beszel
```

Переходим в веб-интерфейс сервиса. В браузере снова вводим IP-адрес узла, добавляем двоеточие и порт 32999.
Пример: http://178.125.85.124:32999. Попадаем на главную страницу входа в сервис. Здесь потребуется либо регистрация, либо быстрый вход через учетную запись GitHub.

После входа в систему найдите на панели иконку выбора языка интерфейса и выберите удобный для вас язык. Нажмите кнопку «Добавить систему» в правом верхнем углу, чтобы открыть диалоговое окно создания новой системы.

Копируем `Ключ` и закрываем окно. Переходим в каталог `beszel` и редактируем файл `compose.yml`.

```sh
cd beszel/
```

```sh
nano compose.yml
```

`Файл compose.yml`

```
version: "3.7"
services:
  beszel:
    image: henrygd/beszel:latest
    container_name: beszel
    restart: unless-stopped
    extra_hosts:
      - host.docker.internal:host-gateway
    ports:
      - "32999:8090"
    volumes:
      - ./beszel_data:/beszel_data

#  beszel-agent:
#    image: henrygd/beszel-agent:latest
#    container_name: beszel-agent
#    restart: unless-stopped
#    network_mode: host
#    volumes:
#      - /var/run/docker.sock:/var/run/docker.sock:ro
#    environment:
#      PORT: 45876
#      KEY: ""
```

Добавляем ключ в кавычки, и удаляем все знаки `#` с блока кода beszel-agent.

`Итоговый файл compose.yml`

```
version: "3.7"
services:
  beszel:
    image: henrygd/beszel:latest
    container_name: beszel
    restart: unless-stopped
    extra_hosts:
      - host.docker.internal:host-gateway
    ports:
      - "32999:8090"
    volumes:
      - ./beszel_data:/beszel_data

  beszel-agent:
    image: henrygd/beszel-agent:latest
    container_name: beszel-agent
    restart: unless-stopped
    network_mode: host
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    environment:
      PORT: 45876
      KEY: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIAcZKcY94gOrPPLTV9Yux74MCD"
```

`Сохраняем итоговый файл compose.yml, нажав клавиши Ctrl + O. Вводим имя файла — compose.yml — и подтверждаем сохранение` 

И запускаем: 

```sh
docker compose up -d
```
`Команда ниже покажет запущенные контейнеры. Ищем колонку STATUS, где должно отображаться состояние UP (healthy)`

```sh
docker ps -a | grep beszel
```

Будет запущено два контейнера: один — сервер, а другой — агент. Снова переходим в веб-интерфейс сервиса Beszel. Нажимаем кнопку «Добавить систему» в правом верхнем углу, чтобы открыть диалоговое окно добавления агента.
Добавляем следующие данные:

- Имя
- IP адрес узла

И нажимаем кнопку `Добавить систему`. Агент добавлен к серверу и начинает получать информацию о системе.

### Настройка уведомлений через Telegram

На главной странице на панели нажимаем на шестеренку (Настройки) и переходим на вкладку Уведомления. Здесь можно настроить уведомления по электронной почте или добавить Webhook-уведомления. Beszel использует Shoutrrr для интеграции с популярными сервисами уведомлений. [Документация Shoutrrr по добавлению Telegram](https://containrrr.dev/shoutrrr/v0.8/services/telegram/). Также доступны интеграции с другими сервисами.

Вот строка, `telegram://token@telegram/?channel-1[,chat-id-1,...]&notification=no&preview=false&parseMode=html`

которую мы должны добавить в сервис Beszel, но сначала добавим в нее свои данные. 

Параметры:

1. Токен (token) - уникальный API токен для взаимодействия с ботом Telegram 
2. Чаты (channels) — идентификаторы чатов или названия каналов 
3. Уведомление (notification) - если отключено, сообщение отправляется автоматически
4. Предварительный просмотр (preview) - если этот параметр отключен, для URL-адресов предварительный просмотр веб-страницы отображаться не будет
5. Режим вывода сообщения (ParseMode) — как следует анализировать текстовое сообщение. Значения: Нет, Markdown, HTML, MarkdownV.2

Токен можно получить при создании, или уже у созданного бота обратившись к боту в телеграм [BotFather](https://telegram.me/BotFather). 

### (Опционально) Создаем бота и получаем API токен

- Откройте приложение Telegram на вашем устройстве. 
- В поиске введите @BotFather, выберите бота с иконкой гаечного ключа и надписью Verified (это официальный бот). Напишите команду /start в чате с BotFather. 
- Затем отправьте команду /newbot. BotFather попросит вас ввести имя вашего бота (это имя, которое будут видеть пользователи). Например, MyTestBot. После этого нужно будет придумать username для вашего бота. Он должен заканчиваться на bot (например, MyTestBot_bot). Этот username должен быть уникальным.
- После успешного создания бота BotFather отправит вам сообщение с API токеном. Он выглядит как строка цифр и букв, например: 123456789:ABCdefGhIJKlmNoPQRstuVWXyz. 

С помощью @BotFather вы можете дополнительно настроить бота:

- Установить описание бота командой /setdescription.
- Добавить картинку профиля командой /setuserpic.
- Настроить команды бота с помощью /setcommands.

Итак получили токен, добавляем в строку, итог: `telegram://123456789:ABCdefGhIJKlmNoPQRstuVWXyz@telegram/?channels=channel&notification=no&preview=false&parseMode=html`

#### Добавляем чат 

Создаем канал или группу в Telegram. Они могут быть как приватными, так и публичными. Добавляем в канал/группу своего бота. Запускаем контейнер на сервере через терминал: 

```sh
docker run --rm -it containrrr/shoutrrr generate telegram
```

И отвечаем на вопросы.

- Вводим свой API токен бота
- Пишем тестовое сообщение в чат с ботом
- Получаем ID чата, например: `-1002321991729`
- Если 1 чат, то отвечаем - нет (no), если более одного то да (yes), и выполняем все действия снова.

Добавляем полученный чат ID в строку, итог: `telegram://123456789:ABCdefGhIJKlmNoPQRstuVWXyz@telegram/?channels=-1002321991729&notification=no&preview=false&parseMode=html`

Редактируем остальные параметры по желанию, и итоговую строку добавляем в Настройки - Уведомления - Webhook. 

Нажимаем кнопку «Тест URL» и получаем тестовое сообщение для проверки корректности настройки. Для настройки уведомлений от сервера переходим на главную страницу, нажимаем на «Колокольчик» и настраиваем соответствующие тумблеры.
