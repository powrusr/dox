# networking

## netcat

### test port is open

```bash
nc -zv 127.0.0.1 8888
Connection to 127.0.0.1 8888 port [tcp/*] succeeded!

# you can use that as a test for something else
# like ssh when the port is alive
if nc -zv 192.168.0.101 22; then ssh host1; fi
```

## nmap

[nmap docs](https://nmap.org/book/toc.html)
[nmap options](https://nmap.org/book/man-briefoptions.html)

### exclude hosts

```bash
sudo nmap -O -A --max-rate 100 --exclude 192.168.0.129,192.168.0.189,192.168.0.240,192. 192.168.0.2-254 -Pn
```

### exclude ports

```bash
nmap 24.0.0.1\24 --exclude-ports 80
```
## bettercap

[bettercap](https://www.bettercap.org/)

The Swiss Army knife for WiFi, Bluetooth Low Energy, wireless HID hijacking and IPv4 and IPv6 networks reconnaissance and MITM attacks

### install

```bash
sudo apt update
sudo apt install golang git build-essential libpcap-dev libusb-1.0-0-dev libnetfilter-queue-dev
```

### usage

[interactive mode](https://www.bettercap.org/usage/interactive/)

### spoofing

```bash
# launch bettercap
bettercap

# enable full duplex arp spoofing
# so target thinks you're the router
# and router thinks you're target
set arp.spoof.fullduplex true

set arp.spoof.targets 192.168.1.111
arp.spoof on

or 

sudo bettercap -eval "set arp.spoof.targets 192.168.1.20; arp.spoof on"

# sniff for traffic
net.sniff on

# may need to enable IP forwarding if target has no internet
echo 1 > /proc/sys/net/ipv4/ip_forward
```
