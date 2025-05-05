# Start the WireGuard VPN server with web interface

![LOGO2](../img/2.png)

[WireGuard - can be found here ](https://goo.su/xKp8kj)

You need a VPS/VDS server to run it. Server can be found [here](https://goo.su/IsDeUO), or [here - for less than 3$ per month, or for a week - for less than 1$](https://vdska.ru/#vds_calc). The operating system is Ubuntu 20 or newer. To run a VPN server in a container you only need 1 core, 1 gigabyte of RAM and from 15 gigabytes of free space. After purchasing the server, you will be given credentials to log in to the server. How to log in to the terminal on the server can be found [here](https://goo.su/BBH9F).

# Table of Contents

  - [Install git, curl, nano, and Docker](#install-git-curl-nano-and-docker)
  - [Edit the compose.yml file to start the VPN server](#edit-the-composeyml-file-to-start-the-vpn-server)
  - [Generate password hash](#generate-a-hash-for-the-password)
  - [Get your routable IP address](#get-your-routable-ip-address)
  - [(Optional) Edit variables in compose.yml file](#optional-edit-variables-in-the-composeyml-file)
  - [Final compose.yml file](#final-file-composeyml)
  - [Add rules to the iptables firewall and enable IP Forward](#add-rules-to-the-iptables-firewall-and-enable-ip-forward)
  - [Start the server](#start-the-server)
  - [Launch the web interface](#launch-the-web-interface)
- [(Optional) Start Beszel monitoring with status notification in Telegram](#optional-start-beszel-monitoring-with-status-notification-in-telegram)
  - [Setting up notifications via Telegram](#setting-up-notifications-via-telegram)
  - [(Optional) Create a bot and get an API token](#optional-create-a-bot-and-get-api-token)


## Install git, curl, nano and Docker

Install git

```sh
sudo apt install -y git
```
Clone the repository
```sh
git clone https://gitea.com/sergeybezlepkin/vpn-wireguard.git
```
Go to the vpn-wireguard directory.
```sh
cd vpn-wireguard/
```
Add execute permissions to the install_soft file and run it
```sh
chmod +x install_soft
```
```sh
./install_soft
```

When installing and configuring the `iptables-persistent` package, you will need to answer the question: `Save current IPv4 rules? [yes/no]` answer `yes`, and to the second question: `Save current IPv6 rules? [yes/no]` answer `yes`.

> !After installation the server will be restarted!

## Edit the compose.yml file to start the VPN server
```sh
cd vpn-wireguard/
```
```sh
nano compose.yml
```

`File compose.yml`

```sh
version: “3.7”
services:
  wg-easy:
    image: ghcr.io/wg-easy/wg-easy
    container_name: wg-easy
    restart: unless-stopped
    volumes:
      - etc_wireguard:/etc/wireguard
    environment:
      - LANG=en
      - WG_HOST=
      - PASSWORD_HASH=
      - UI_TRAFFIC_STATS=true
      - UI_CHART_TYPE=3
      - WG_PORT=49888
      - PORT=48999
      - WG_DEFAULT_ADDRESS=10.45.0.x
      - WG_DEFAULT_DNS=1.1.1.1.1, 8.8.8.8.8
      - WG_ALLOWED_IPS=0.0.0.0.0/0, ::/0
      - WG_PERSISTENT_KEEPALIVE=25
      # - WG_PRE_UP=echo “Pre Up” > /etc/wireguard/pre-up.txt
      # - WG_POST_UP=echo “Post Up” > /etc/wireguard/post-up.txt
      # - WG_PRE_DOWN=echo “Pre Down” > /etc/wireguard/pre-down.txt
      # - WG_POST_DOWN=echo “Post Down” > /etc/wireguard/post-down.txt
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

` Leave the terminal window open. It will need to be updated with your data. Open the second tab in the terminal. `

## Generate a hash for the password

```sh
cd vpn-wireguard/
```
```sh
cat hash
```
```sh
docker run --rm -it ghcr.io/wg-easy/wg-easy wgpw 'YOUR_PASSWORD'
```
After downloading and running the image, the terminal will display the HASH string write 
> PASSWORD_HASH='$2a$12$o1s1n4ruKZ/2RbnVOUoHOutLY6raO4IDZv7/5z/Qh.K0UbA8RyaVG''.

Copy this line and paste it into the `PASSWORD_HASH=` environment variable in the `compose.yml` file. It should be added after the equal sign and without quotation marks. There are $ characters in the hash string, add one additional $ character next to each existing one. 

We get the total: `PASSWORD_HASH=$$$2a$$12$$$o1s1n4ruKZ/2RbnVOUoHOutLY6raO4IDZv7/5z/Qh.K0UbA8RyaVG`.

## Get your routable IP address

```sh
curl ifconfig.me
```
`or`
```sh
curl ipinfo.io
```

Add the resulting address string to the `WG_HOST=` environment variable after the equal sign and without quotes.

We get: `- WG_HOST=178.125.85.124`. 

## (Optional) Edit variables in the compose.yml file

- In the `LANG=ru` environment variable you can change the interface language. Available options: en, ua, ru, tr, no, pl, fr, de, ca, es, ko, vi, nl, is, pt, chs, cht, it, th, hi, ja, si.
- In the `PORT=48999` and `WG_PORT` environment variable, you can specify a convenient port. For example, select a free port from the ranges 48658-48999 or 49001-49150.
- In the `WG_DEFAULT_ADDRESS=10.45.0.x` environment variable, you can change the second octet of the address to 8 or 6. For example, you would get 10.8.0.x or 10.6.0.x.
- After making the necessary changes, save the compose.yml file.

## Final file compose.yml

```sh
version: “3.7”
services:
  wg-easy:
    image: ghcr.io/wg-easy/wg-easy
    container_name: wg-easy
    restart: unless-stopped
    volumes:
      - etc_wireguard:/etc/wireguard
    environment:
      - LANG=en
      - WG_HOST=178.125.85.124
      - PASSWORD_HASH=$$$2a$$12$$$o1s1n4ruKZ/2RbnVOUoHOutLY6raO4IDZv7/5z/Qh.K0UbA8RyaVG
      - UI_TRAFFIC_STATS=true
      - UI_CHART_TYPE=3
      - WG_PORT=49888
      - PORT=48999
      - WG_DEFAULT_ADDRESS=10.45.0.x
      - WG_DEFAULT_DNS=1.1.1.1.1, 8.8.8.8.8
      - WG_ALLOWED_IPS=0.0.0.0.0/0, ::/0
      - WG_PERSISTENT_KEEPALIVE=25
      # - WG_PRE_UP=echo “Pre Up” > /etc/wireguard/pre-up.txt
      # - WG_POST_UP=echo “Post Up” > /etc/wireguard/post-up.txt
      # - WG_PRE_DOWN=echo “Pre Down” > /etc/wireguard/pre-down.txt
      # - WG_POST_DOWN=echo “Post Down” > /etc/wireguard/post-down.txt
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

`Save the final file compose.yml by pressing Ctrl + O. Enter the name of the file - compose.yml - and confirm saving `

## Add rules to the iptables firewall and enable IP Forward.

```sh
echo net.ipv4.ip_forward = 1 >> /etc/sysctl.conf
```
```sh
sudo sysctl -p
```

To add rules to iptables properly, you must specify the name of your network interface.

Output the list of interfaces, state, address:

```sh
ip -br a
```
Get:
| `Interface` | Status | IPv4/IPv6 address |
|---------------|--------|----------------------------------|
| lo | UNKNOWN| 127.0.0.1.1/8 ::1/128 |
| `enp0s3` | UP | 192.168.1.12/24 fe80::a00:27ff:fe90:fe8d/64 |
| docker0 | DOWN | 172.17.0.1/16 fe80::42:abff:febb:86ce/64 |
| docker_gwbridge | UP | 172.20.0.1/16 fe80::42:31ff:fe46:73b3/64 |
| veth4550803@if8 | UP | fe80::a404:7ff:fed5:585e/64 |
| veth8d014ec@if22 | UP | fe80::1ca2:29ff:feee:9a00/64 |

Now add commands to the terminal: 
```sh
sudo iptables -t nat -I POSTROUTING 1 -s 10.8.0.0.0/24 -o YOUR_INTERFACE -j MASQUERADE
```
where, `10.8.0.0.0/24` is your IP address from the environment variable `- WG_DEFAULT_ADDRESS=10.8.0.x` and `eth0` is the name of your interface.
Example:
```sh.
sudo iptables -t nat -I POSTROUTING 1 -s 10.45.0.0.0/24 -o enp0s3 -j MASQUERADE
```
Next, you need to substitute your interface and apply each command:

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
sudo iptables -A FORWARD -i wg0 -o wg0 -j ACCEPT
```
```sh
sudo iptables -I INPUT 1 -i YOUR_INTERFACE -p udp --dport 49888 -j ACCEPT
```

> The installed iptables-persistent package allows you to save iptables rules after a server reboot.

```sh
sudo netfilter-persistent save
```
```sh
sudo systemctl enable netfilter-persistent
```
```sh
sudo systemctl start netfilter-persistent
```

## Start the server

```sh
cd vpn-wireguard/
```
```sh
docker compose up -d
```
`` The command below will show the running containers. Look for the STATUS column, which should show the UP (healthy)``

```sh
docker ps -a | grep wg-easy
```

## Launch the web interface

![LOGO2](../img/1.PNG)

Go to the browser and enter the IP address of the host from the `WG_HOST=XXX.XXX.XXX` variable, add a colon and the port number from the `PORT=XXXX` variable. Пример: http://178.125.85.124:48999.
We get to the main page with a password input field. Enter the password that was created using hash string. After successful login, the interface will open. Click the “+ Create” or “+ Create Client” button, enter the name of the client. The configuration will be ready. 

Each client has:

- A toggle switch to turn on/off.
- A QR code button for scanning.
- A button to download the configuration.
- A button to uninstall the client.

![LOGO2](../img/2.png)

#### [Download client for your devices](https://www.wireguard.com/install/)

Use the nmap package to check if the open port (specified in the WG_PORT variable) is available from the internet.

```sh
sudo nmap -sU -p 49888 $(curl ifconfig.me)
```
There will be an entry 

```sh
PORT STATE SERVICE
49888/udp open|filtered unknown
Nmap done: 1 IP address (1 host up) scanned in 2.40 seconds
```

The OPEN state indicates that the port is available and accepting connections.

### (Optional) Start Beszel monitoring with status notification in Telegram

[Beszel can be found here ](https://beszel.dev/) 

```sh
cd beszel
```
Run: 

```sh
docker compose up -d
```

The command below will show the running container, look for the `STATUS` column to see the `UP (healthy)` state.

```sh
docker ps -a | grep beszel
```

Go to the web interface of the service. In the browser, re-enter the IP address of the host, add a colon and port 32999. Example: http://178.125.85.124:32999. We get to the main page of logging into the service. Here you will need either registration or a quick login via your GitHub account.

![LOGO3](../img/3.png)

Once logged in, find the interface language selection icon on the panel and choose a language you are comfortable with. Click the `Add System` button in the upper right corner to open the Create New System dialog box.

Copy the `key` and close the dialog box. Go to the `beszel `directory and edit the `compose.yml` file.

```sh
cd beszel/
```

```sh
nano compose.yml
```

`File compose.yml`

```sh
version: “3.7”
services:
  beszel:
    image: henrygd/beszel:latest
    container_name: beszel
    restart: unless-stopped
    extra_hosts:
      - host.docker.internal:host-gateway
    ports:
      - “32999:8090”
    volumes:
      - ./beszel_data:/beszel_data

# beszel-agent:
# image: henrygd/beszel-agent:latest
# container_name: beszel-agent
# restart: unstopped
# network_mode: host
# volumes:
# - /var/run/docker.sock:/var/run/docker.sock:ro
# environment:
# PORT: 45876
# KEY: “”
```

Add the key in quotes, and remove all `#` signs from the beszel-agent code block.

`The final file is compose.yml`

```sh
version: “3.7”
services:
  beszel:
    image: henrygd/beszel:latest
    container_name: beszel
    restart: unless-stopped
    extra_hosts:
      - host.docker.internal:host-gateway
    ports:
      - “32999:8090”
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
      KEY: “ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAAAIAcZKcY94gOrPPLTV9Yux74MCD”
```

`Save the final file compose.yml by pressing Ctrl + O. Enter the name of the file - compose.yml - and confirm the save ` 

And run: 

```sh
docker compose up -d
```
`The command below will show the running containers. Look for the STATUS column, where the UP (healthy) should be displayed`

```sh
docker ps -a | grep beszel
```

Two containers will be running, one is the server and the other is the agent. Go to the web interface of the Beszel service again. Click the “Add System” button in the upper right corner to open the Add Agent dialog box.
We add the following information:

- Name
- IP address of the host

![LOGO4](../img/4.png)

And click the `Add System` button. The agent is added to the server and starts receiving information about the system.

### Setting up notifications via Telegram

On the home page, click on the gear (Settings) on the panel and go to the Notifications tab. Here you can set up email notifications or add Webhook notifications. Beszel uses Shoutrrrr to integrate with popular notification services. [Shoutrrrr documentation on adding Telegram](https://containrrr.dev/shoutrrr/v0.8/services/telegram/). Integrations with other services are also available.

Here's the string, `telegram://token@telegram/?channel-1[,chat-id-1,...]&notification=no&preview=false&parseMode=html`

which we should add to the Beszel service, but first let's add our data to it. 

Parameters:

1. token - a unique API token for interaction with Telegram bot 
2. Chats (channels) - chat identifiers or channel names 
3. Notification (notification) - if disabled, the message is sent automatically
4. Preview - If this option is disabled, no preview of the web page will be displayed for URLs
5. Message output mode (ParseMode) - How the text message should be parsed. Values: None, Markdown, HTML, MarkdownV.2

Token can be obtained at creation or from already created bot by contacting bot in Telegram [BotFather](https://telegram.me/BotFather). 

### (Optional) Create a bot and get API token

- Open the Telegram app on your device. 
- Type @BotFather in search, select the bot with the wrench icon and the Verified label (this is the official bot). Write the /start command in chat with BotFather. 
- Then send the command /newbot. BotFather will ask you to enter the name of your bot (this is the name that users will see). For example, MyTestBot. After that, you will need to come up with a username for your bot. It should end with bot (for example, MyTestBot_bot). This username must be unique.
- After successfully creating the bot, BotFather will send you a message with an API token. It looks like a string of numbers and letters, for example: 123456789:ABCdefGhIJKlmNoPQRstuVWXyz. 

With @BotFather, you can further customize the bot:

- Set bot description with the /setdescription command.
- Add a profile picture with the /setuserpic command.
- Customize bot commands with /setcommands.

So we got the token, add it to the string, total: `telegram://123456789:ABCdefGhIJKlmNoPQRstuVWXyz@telegram/?channels=channel&notification=no&preview=false&parseMode=html`.

#### Add chat 

Create a channel or group in Telegram. They can be either private or public. Add your bot to the channel/group. Run the container on the server via terminal: 

```sh
docker run --rm -it containrrrrr/shoutrrrr generate telegram
```

And answer the questions.

- Enter your API bot token
- Write a test message to the chat with the bot.
- Get the chat ID, for example: `-1002321991729`
- If there is 1 chat, we answer no (no), if there is more than one chat, we answer yes (yes), and perform all actions again.

Add the received chat ID to the string, total: `telegram://123456789:ABCdefGhIJKlmNoPQRstuVWXyz@telegram/?channels=-1002321991729&notification=no&preview=false&parseMode=html`.

Edit the rest of the parameters as desired, and add the final string to Settings - Notifications - Webhook.

![LOGO5](../img/5.png)

Click the “Test URL” button and get a test message to check if the settings are correct. To customize notifications from the server, go to the main page, click on “Bell” and customize the corresponding toggle switches.
