# Tp7

### I. Setup

### II. Serveur Web

🌞 Lister les ports en écoute sur la machine

````
[wheelroot@web ~]$ sudo ss -lnpt | grep 80
LISTEN 0      511          0.0.0.0:80        0.0.0.0:*    users:(("nginx",pid=766,fd=6),("nginx",pid=765,fd=6))
LISTEN 0      511             [::]:80           [::]:*    users:(("nginx",pid=766,fd=7),("nginx",pid=765,fd=7))
````

🌞 Ouvrir le port dans le firewall de la machine

````
[wheelroot@web ~]$ sudo firewall-cmd --permanent --add-port=80/tcp
Warning: ALREADY_ENABLED: 80:tcp
success
[wheelroot@web ~]$ sudo firewall-cmd --reload
success
````


🌞 Vérifier que ça a pris effet

````
[wheelroot@client1 ~]$ ping sitedefou.tp7.b1
PING sitedefou.tp7.b1 (10.7.1.11) 56(84) bytes of data.
64 bytes from sitedefou.tp7.b1 (10.7.1.11): icmp_seq=1 ttl=64 time=1.42 ms
64 bytes from sitedefou.tp7.b1 (10.7.1.11): icmp_seq=2 ttl=64 time=1.28 ms
64 bytes from sitedefou.tp7.b1 (10.7.1.11): icmp_seq=3 ttl=64 time=0.351 ms
64 bytes from sitedefou.tp7.b1 (10.7.1.11): icmp_seq=4 ttl=64 time=0.832 ms
^C
--- sitedefou.tp7.b1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 0.351/0.970/1.422/0.418 ms
````


````
[wheelroot@client1 ~]$ curl http://sitedefou.tp7.b1
meow!
````



🌞 Lister les ports en écoute sur la machine

````
[wheelroot@web ~]$ sudo ss -lnpt | grep 443
LISTEN 0      511        10.7.1.11:443       0.0.0.0:*    users:(("nginx",pid=1309,fd=6),("nginx",pid=1308,fd=6))
````

🌞 Gérer le firewall

````
[wheelroot@web ~]$ sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3
  sources:
  services: cockpit dhcpv6-client ssh
  ports: 443/tcp
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
````




### III. Serveur VPN


🌞 Prouvez que vous avez bien une nouvelle carte réseau wg0
````
[wheelroot@vpn ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:d3:b3:5f brd ff:ff:ff:ff:ff:ff
    inet 10.7.1.111/24 brd 10.7.1.255 scope global noprefixroute enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fed3:b35f/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:8e:d6:bb brd ff:ff:ff:ff:ff:ff
    inet 10.7.2.111/24 brd 10.7.2.255 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe8e:d6bb/64 scope link
       valid_lft forever preferred_lft forever
4: wg0: <POINTOPOINT,NOARP,UP,LOWER_UP> mtu 1420 qdisc noqueue state UNKNOWN group default qlen 1000
    link/none
    inet 10.7.200.1/24 scope global wg0
       valid_lft forever preferred_lft forever
````

🌞 Déterminer sur quel port écoute Wireguard

````
[wheelroot@vpn ~]$ sudo ss -lnpu | grep 51820
UNCONN 0      0            0.0.0.0:51820      0.0.0.0:*
UNCONN 0      0               [::]:51820         [::]:*
````

🌞 Ouvrez ce port dans le firewall

````
[wheelroot@vpn ~]$ sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8 wg0
  sources:
  services: cockpit dhcpv6-client ssh
  ports: 22/tcp 51820/udp
  protocols:
  forward: yes
  masquerade: yes
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
````

🌞 Ping ping ping !

````
[wheelroot@client1 ~]$ ping 10.7.200.1
PING 10.7.200.1 (10.7.200.1) 56(84) bytes of data.
64 bytes from 10.7.200.1: icmp_seq=1 ttl=64 time=1.69 ms
64 bytes from 10.7.200.1: icmp_seq=2 ttl=64 time=2.27 ms
64 bytes from 10.7.200.1: icmp_seq=3 ttl=64 time=0.596 ms
64 bytes from 10.7.200.1: icmp_seq=4 ttl=64 time=0.793 ms
^C
--- 10.7.200.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3007ms
rtt min/avg/max/mdev = 0.596/1.338/2.271/0.679 ms
````

🌞 Capture ping1_vpn.pcap


🌞 Capture ping2_vpn.pcap


🌞 Prouvez que vous avez toujours un accès internet

````
[wheelroot@client1 ~]$ traceroute 1.1.1.1
traceroute to 1.1.1.1 (1.1.1.1), 30 hops max, 60 byte packets
 1  _gateway (10.7.200.1)  3.386 ms  3.371 ms  3.362 ms
 2  * * *
 3  10.0.2.2 (10.0.2.2)  13.925 ms  13.928 ms  14.460 ms
 4  * * *
 5  * * *
 6  * * *
 7  * * *
 8  * * *
 9  *
 ````

 4. Private service

 🌞 Visitez le service Web à travers le VPN

 ````
[wheelroot@client1 ~]$ curl -k https://sitedefou.tp7.b1
meow!
````