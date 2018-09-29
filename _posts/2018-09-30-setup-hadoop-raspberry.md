---
layout: post
---
This post presents guidelines to configure network for Raspberry Pi Model 2/3 inside console environment.
## Setting up Wifi connection
* All the network interfaces are contained in ```/etc/network//interfaces```, change it with vi/vim editor.

```console
sudo vi /etc/network/interfaces
```

* To automatically connect to wifi when raspberry boot,
add the following into the *first line*.

```console
auto wlan0
```

* Add these lines at the bottom to tell Raspberry Pi to use the ```/etc/wpa_supplicant/wpa_supplicant.conf``` as your configuration file,  this will connect Raspberry Pi with dynamic ip via dhcp server.

```console
allow-hotplug wlan0
iface wlan0 inet dhcp
wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
iface default inet dhcp
```

* With your router IP of ```192.168.0.1```, to connect with static IP ```192.168.0.100```, add these lines instead.

```console
allow-hotplug wlan0
iface wlan0 inet static
iface wlan0 inet static
address 192.168.0.100
netmask 255.255.255.0
gateway 192.168.0.1
wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
```

* To configure network id and password, open up ```/etc/wpa_supplicant/wpa_supplicant.conf```

```console
sudo vi /etc/wpa_supplicant/wpa_supplicant.conf
```

* ```ssid``` and ```psk``` is needed as minimum, if your network requires more information, add accordingly. Multiple blocks of ```network``` can be configured for various wifi networks.

```console
network={
ssid="YOUR_NETWORK_NAME"
psk="YOUR_NETWORK_PASSWORD"
proto=RSN
key_mgmt=WPA-PSK
pairwise=CCMP
auth_alg=OPEN
}
```

## Setting up Local Area Network (LAN) with static IP address
* Edit the file ```/etc/dhcpcd.conf``` as following.
```sudo vi /etc/dhcpcd.conf```

```console
interface eth0
static ip_address=192.168.0.100/24
static routers=192.168.0.1
static domain_name_servers=192.168.0.1
```

## Trouble shooting
* Manually activate the LAN network interface by.

```console
sudo ifup eth0
```
* After changing the Wifi details, run the following command to reconfigure the network

```console
sudo wpa_cli -i wlan0 reconfigure
```

* Reboot the Raspberry Pi once the setup is completed.
