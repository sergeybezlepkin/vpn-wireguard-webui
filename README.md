## Запускаем свой выделенный VPN-сервер WireGuard c web-интерфейсом

[О WireGuard можно узнать здесь ](https://goo.su/xKp8kj)

## VPS/VDS

Для запуска нужен VPS/VDS сервер. Сервер можно найти [тут](https://goo.su/YkTH0), или [вот тут меньше чем за 3$ в месяц, или на неделю меньше чем за 1$](https://vdska.ru/#vds_calc). Операционная система Ubuntu 20/Debian 10 или новее.

# Ubuntu (рекомендую)

## Установливаем git, curl, nano и Docker

### Устанавливаем git

```sh
sudo apt install -y git
```
### Клонируем репозиторий
```sh
git clone https://github.com/sergeybezlepkin/vpn-wireguard.git
```
### Переходим в каталог vpn-wireguard
```sh
cd vpn-wireguard/
```
### Добавляем права на выполнения файла install_soft и запускаем
```sh
chmod +x install_soft
```
```sh
./install_soft
```
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
      - WG_PORT=51820
      - PORT=48999
      - WG_DEFAULT_ADDRESS=10.45.0.x
      - WG_DEFAULT_DNS=1.1.1.1, 8.8.8.8
      - WG_ALLOWED_IPS=10.45.0.0/24
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

`Оставляем открытым в терминале, нужно дополнить своими данными. Открываем вторую вкладку в терминале.`

## Генерируем хэш для пароля

```sh
cat hash.txt
```
```sh
docker run --rm -it ghcr.io/wg-easy/wg-easy wgpw 'YOUR_PASSWORD'
```
После загрузки и запуска образа, терминал выдаст HASH-строку, записываем 
> PASSWORD_HASH='$2a$12$o1s1n4ruKZ/2RbnVOUoHOutLY6raO4IDZv7/5z/Qh.K0UbA8RyaVG'

Копируем полученную строку, в переменную окружения `- PASSWORD_HASH=` в файле compose.yml. А добавить ее надо после равно и без ковычек. В HASH-строке есть символы `$`, добавляем еще по одному рядом с тем, с которым получили от запуска образа.
Получаем итог: `PASSWORD_HASH=$$2a$$12$$o1s1n4ruKZ/2RbnVOUoHOutLY6raO4IDZv7/5z/Qh.K0UbA8RyaVG`

## Узнаем свой маршрутизируемый IP-адрес

```sh
curl ifconfig.me
```
`или`
```sh
curl ipinfo.io
```

Полученную строку с адресом, добавляем в переменную окружения `- WG_HOST=` после равно и без ковычек. 

Получаем: `- WG_HOST=178.125.85.124` 

#### Итоговый файл compose.yml
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
      - WG_HOST=178.125.85.124
      - PASSWORD_HASH=$$2a$$12$$o1s1n4ruKZ/2RbnVOUoHOutLY6raO4IDZv7/5z/Qh.K0UbA8RyaVG
      - UI_TRAFFIC_STATS=true
      - UI_CHART_TYPE=3
      - WG_PORT=51820
      - PORT=48999
      - WG_DEFAULT_ADDRESS=10.45.0.x
      - WG_DEFAULT_DNS=1.1.1.1, 8.8.8.8
      - WG_ALLOWED_IPS=10.45.0.0/24
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

## Правим переменные в файле compose.yml под свои нужды

В переменной окружения `- PORT=48999` можно изменить удобный для себя порт с 48658—48999, или с 49001—49150. 
В переменной окружения `- WG_DEFAULT_ADDRESS=10.45.0.x` можно изменить вторую октету адреса, на 8 или 6. Получаем: `10.8.0.x` или `10.6.0.x`. После внесения нужных данных, сохраняем файл compose.yml.

## Добавляем правила в брандмауэр iptables

Для правильного добавления правил нужно вписать в комманду имя своего интерфейса.
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
где, `10.8.0.0/24` - свой IP адрес из переменной окружения `- WG_DEFAULT_ADDRESS=10.8.0.x` И `eth0` имя своего интерфейса.
Пример:
```sh
sudo iptables -t nat -I POSTROUTING 1 -s 10.45.0.0/24 -o enp0s3 -j MASQUERADE
```
Даллее, по аналогии нужно подставить свой интерфейс и применить каждую комманду:

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
sudo iptables -I INPUT 1 -i YOUR_INTERFACE -p udp --dport 51820 -j ACCEPT
```

## Запускаем сервер

```sh
cd vpn-wireguard/
```
```sh
docker compose up -d 
```

Проверяем запущенный контейнер
```sh
docker ps -a | grep wg-easy
```
