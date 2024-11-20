## Setup

Install Raspbian (lite) on a pi (ensure it has a wifi adapter)

Run the following command to install required packages...

```bash
sudo apt-get -y install dhcpcd iptables dnsmasq
```

### Configure `dnsmasq`

Edit `/etc/dnsmasq.conf` and add/uncomment the ethernet interface, range of IP addresses to allocate and how often the device can use the address

```
interface=eth0 dhcp-range=192.168.2.50,192.168.2.150,12h
```

Restart the service

```bash
sudo systemctl restart dnsmasq
```

### Enable IP Forwarding

Edit `/etc/sysctl.conf` and add/uncomment...

```
net.ipv4.ip_forward=1
```

Reload this config by running...

```bash
sudo sysctl -p
```

### Setup NAT with `iptables`

Add the NAT rules...

```bash
sudo iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
sudo iptables -A FORWARD -i eth0 -o wlan0 -j ACCEPT
sudo iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT
```

And the persist rules...

```bash
sudo apt install iptables-persistent sudo sh -c "iptables-save > /etc/iptables/rules.v4"
```

## Troubleshooting

Ensure that `eth0` has the correct IP by running `ifconfig eth0`. The output should show `inet` with a static IP...

```
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.2.1  netmask 255.255.255.0  broadcast 192.168.2.255
        inet6 fe80::b180:9d5f:6c7d:15d4  prefixlen 64  scopeid 0x20<link>
        ether b8:27:eb:24:45:c8  txqueuelen 1000  (Ethernet)
        RX packets 134  bytes 17270 (16.8 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 90  bytes 10963 (10.7 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
