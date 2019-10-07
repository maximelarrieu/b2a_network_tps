# TP2 : Network low-level, Switching

## I. Simplest setup

Topologie mise en place :

```
+-----+        +-------+        +-----+
| PC1 +--------+  SW1  +--------+ PC2 |
+-----+        +-------+        +-----+
```

Ping de PC1 vers PC2 :

```
$ ping 10.2.1.1
PING 10.2.1.1 (10.2.1.1) 56(84) bytes of data.
64 bytes from 10.2.1.1: icmp_seq=1 ttl=64 time=5.01 ms
64 bytes from 10.2.1.1: icmp_seq=2 ttl=64 time=5.81 ms
64 bytes from 10.2.1.1: icmp_seq=3 ttl=64 time=4.75 ms
64 bytes from 10.2.1.1: icmp_seq=4 ttl=64 time=5.35 ms
64 bytes from 10.2.1.1: icmp_seq=5 ttl=64 time=3.61 ms
^C
--- 10.2.1.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 11ms
rtt min/avg/max/mdev = 3.608/4.905/5.801/0.738 ms
```

Analyse Wireshark du ping de PC2 vers PC1

```
31	23.916701	10.2.1.2	10.2.1.1	ICMP	98	Echo (ping) request  id=0x06bf, seq=7/1792, ttl=64 (reply in 32)
32	23.920234	10.2.1.1	10.2.1.2	ICMP	98	Echo (ping) reply    id=0x06bf, seq=7/1792, ttl=64 (request in 31)
34	24.918869	10.2.1.2	10.2.1.1	ICMP	98	Echo (ping) request  id=0x06bf, seq=8/2048, ttl=64 (reply in 35)
35	24.922306	10.2.1.1	10.2.1.2	ICMP	98	Echo (ping) reply    id=0x06bf, seq=8/2048, ttl=64 (request in 34)
36	25.920838	10.2.1.2	10.2.1.1	ICMP	98	Echo (ping) request  id=0x06bf, seq=9/2304, ttl=64 (reply in 37)
37	25.924180	10.2.1.1	10.2.1.2	ICMP	98	Echo (ping) reply    id=0x06bf, seq=9/2304, ttl=64 (request in 36)
```


Le protocole utilisé est ICMP qui signifie Internet Control Message Protocol.

Ce switch ne possède pas d'IP car sa mission est de faire le pont entre le PC1 et le PC2, il n'est qu'un intermédiaire.

Les machines ont besoin d'une IP pour pouvoir se ping et plus généralement pour communiquer car c'est le protocole pour 
discuter à travers le réseau qui est "portée" par une carte réseau. (cf mémo)

## II. More Switches

Topologie mise en place :

```
                        +-----+
                        | PC2 |
                        +--+--+
                           |
                           |
                       +---+---+
                   +---+  SW2  +----+
                   |   +-------+    |
                   |                |
                   |                |
+-----+        +---+---+        +---+---+        +-----+
| PC1 +--------+  SW1  +--------+  SW3  +--------+ PC3 |
+-----+        +-------+        +-------+        +-----+
```

Ping de PC1 vers PC2 :

```
$ ping 10.2.1.2
PING 10.2.1.2 (10.2.1.2) 56(84) bytes of data.
64 bytes from 10.2.1.2: icmp_seq=1 ttl=64 time=3.28 ms
64 bytes from 10.2.1.2: icmp_seq=2 ttl=64 time=3.35 ms
64 bytes from 10.2.1.2: icmp_seq=3 ttl=64 time=3.43 ms
^C
--- 10.2.1.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 6ms
rtt min/avg/max/dev = 3.281/3.353/3.434/0.091 ms
```

Ping de PC1 vers PC3 :

```
```

Ping de PC2 vers PC1 :

```
```
Ping de PC2 vers PC3 :

```
```

Ping de PC3 vers PC1 :

```
```

Ping de PC3 vers PC2 :

```
```

Analyser la table MAC d'un switch :

```
IOU2#show mac address-table 
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    0800.274e.1c9f    DYNAMIC     Et0/2
   1    0800.2782.efcb    DYNAMIC     Et0/1
   1    aabb.cc00.0210    DYNAMIC     Et0/1
   1    aabb.cc00.0300    DYNAMIC     Et0/2
   1    aabb.cc00.0310    DYNAMIC     Et0/1
Total Mac Addresses for this criterion: 5
```

