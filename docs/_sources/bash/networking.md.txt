# networking

## iperf3

do a speed test with one of the publically available remote iperf3 servers from the comfort of you terminal
- [public iperf3 servers](https://iperf3serverlist.net/)

```bash
sudo apt-get install iperf3

# pick one of the public server urls
iperf3 -R -c lg.vie.alwyzon.net -p 5202-5203         
Connecting to host lg.vie.alwyzon.net, port 5202
Reverse mode, remote host lg.vie.alwyzon.net is sending
[  5] local 2a02:aaaa:bbbb:cccc:49dd:brbb:fake:loll port 48898 connected to 2a0d:aaaa:bbbb::6 port 5202
[ ID] Interval           Transfer     Bitrate
[  5]   0.00-1.14   sec   128 KBytes   917 Kbits/sec                  
[  5]   1.14-2.30   sec   128 KBytes   903 Kbits/sec                  
[  5]   2.30-3.46   sec   128 KBytes   908 Kbits/sec                  
[  5]   3.46-4.61   sec   128 KBytes   907 Kbits/sec                  
[  5]   4.61-5.78   sec   128 KBytes   896 Kbits/sec                  
[  5]   5.78-6.86   sec   128 KBytes   979 Kbits/sec                  
[  5]   6.86-7.91   sec   128 KBytes   993 Kbits/sec                  
[  5]   7.91-9.01   sec   128 KBytes   952 Kbits/sec                  
[  5]   9.01-10.14  sec   128 KBytes   929 Kbits/sec                  
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.18  sec  1.37 MBytes  1.13 Mbits/sec   87             sender
[  5]   0.00-10.14  sec  1.12 MBytes   930 Kbits/sec                  receiver
```

without iperf3

```bash
curl -s https://raw.githubusercontent.com/sivel/speedtest-cli/master/speedtest.py | python -
Retrieving speedtest.net configuration...
<stdin>:960: DeprecationWarning: datetime.datetime.utcnow() is deprecated and scheduled for removal in a future version. Use timezone-aware objects to represent datetimes in UTC: datetime.datetime.now(datetime.UTC).
Testing from Telenet (141.111.111.111)...
Retrieving speedtest.net server list...
Selecting best server based on ping...
Hosted by Orange Belgium (Antwerp) [82.18 km]: 37.074 ms
Testing download speed................................................................................
Download: 0.66 Mbit/s
Testing upload speed......................................................................................................
Upload: 0.30 Mbit/s
```


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
