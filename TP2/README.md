# TP2 : Network low-level, Switching

## I. Simplest setup

### Topologie mise en place

```
+-----+        +-------+        +-----+
| PC1 +--------+  SW1  +--------+ PC2 |
+-----+        +-------+        +-----+
```

### Faire communiquer les deux PCs

#### Avec un ping qui fonctionne

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

##### Déterminer le protocole utilisé par ping 

D'après Wireshark, protocole utilisé est ICMP qui signifie Internet Control Message Protocol.

`31	23.916701	10.2.1.2	10.2.1.1	ICMP	98	Echo (ping) request  id=0x06bf, seq=7/1792, ttl=64 (reply in 32)`

Analyse Wireshark du ping de PC2 vers PC1

```
31	23.916701	10.2.1.2	10.2.1.1	ICMP	98	Echo (ping) request  id=0x06bf, seq=7/1792, ttl=64 (reply in 32)
32	23.920234	10.2.1.1	10.2.1.2	ICMP	98	Echo (ping) reply    id=0x06bf, seq=7/1792, ttl=64 (request in 31)
34	24.918869	10.2.1.2	10.2.1.1	ICMP	98	Echo (ping) request  id=0x06bf, seq=8/2048, ttl=64 (reply in 35)
35	24.922306	10.2.1.1	10.2.1.2	ICMP	98	Echo (ping) reply    id=0x06bf, seq=8/2048, ttl=64 (request in 34)
36	25.920838	10.2.1.2	10.2.1.1	ICMP	98	Echo (ping) request  id=0x06bf, seq=9/2304, ttl=64 (reply in 37)
37	25.924180	10.2.1.1	10.2.1.2	ICMP	98	Echo (ping) reply    id=0x06bf, seq=9/2304, ttl=64 (request in 36)
```

### Expliquer...

#### Pourquoi le switch n'a pas besoin d'IP

Ce switch ne possède pas d'IP car sa mission est de faire le pont entre le PC1 et le PC2, il n'est qu'un intermédiaire.

#### Pourquoi les machines ont besoin d'une IP pour pouvoir se ping

Les machines ont besoin d'une IP pour pouvoir se ping et plus généralement pour communiquer car c'est le protocole pour 
discuter à travers le réseau qui est "portée" par une carte réseau. (cf mémo)

## II. More Switches

### Topologie mise en place

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

### Faire communiquer les trois PCs

#### Avec des pings qui fonctionnent

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
$ ping 10.2.1.3
PING 10.2.1.3 (10.2.1.3) 56(84) bytes of data.
64 bytes from 10.2.1.3: icmp_seq=1 ttl=64 time=2.18 ms
64 bytes from 10.2.1.3: icmp_seq=2 ttl=64 time=3.75 ms
64 bytes from 10.2.1.3: icmp_seq=3 ttl=64 time=3.49 ms
^C
--- 10.2.1.3 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 6ms
rtt min/avg/max/dev = 2.214/3.390/1.894/0.022 ms
```

Ping de PC2 vers PC1 :

```
$ ping 10.2.1.1
PING 10.2.1.1 (10.2.1.1) 56(84) bytes of data.
64 bytes from 10.2.1.1: icmp_seq=1 ttl=64 time=1.94 ms
64 bytes from 10.2.1.1: icmp_seq=2 ttl=64 time=3.56 ms
64 bytes from 10.2.1.1: icmp_seq=3 ttl=64 time=2.19 ms
^C
--- 10.2.1.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 6ms
rtt min/avg/max/dev = 3.529/2.221/1.916/0.017 ms
```
Ping de PC2 vers PC3 :

```
$ ping 10.2.1.3
PING 10.2.1.3 (10.2.1.3) 56(84) bytes of data.
64 bytes from 10.2.1.3: icmp_seq=1 ttl=64 time=3.18 ms
64 bytes from 10.2.1.3: icmp_seq=2 ttl=64 time=2.59 ms
64 bytes from 10.2.1.3: icmp_seq=3 ttl=64 time=2.41 ms
^C
--- 10.2.1.3 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 6ms
rtt min/avg/max/dev = 3.621/3.753/2.846/0.071 ms
```

Ping de PC3 vers PC1 :

```
$ ping 10.2.1.1
PING 10.2.1.1 (10.2.1.1) 56(84) bytes of data.
64 bytes from 10.2.1.1: icmp_seq=1 ttl=64 time=2.68 ms
64 bytes from 10.2.1.1: icmp_seq=2 ttl=64 time=3.15 ms
64 bytes from 10.2.1.1: icmp_seq=3 ttl=64 time=3.11 ms
^C
--- 10.2.1.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 6ms
rtt min/avg/max/dev = 2.411/3.366/2.734/0.054 ms
```

Ping de PC3 vers PC2 :

```
$ ping 10.2.1.2
PING 10.2.1.2 (10.2.1.2) 56(84) bytes of data.
64 bytes from 10.2.1.2: icmp_seq=1 ttl=64 time=1.76 ms
64 bytes from 10.2.1.2: icmp_seq=2 ttl=64 time=2.29 ms
64 bytes from 10.2.1.2: icmp_seq=3 ttl=64 time=3.13 ms
^C
--- 10.2.1.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 6ms
rtt min/avg/max/dev = 2.818/3.123/3.498/0.176 ms
```

### Analyser la table MAC d'un switch :

#### `show mac address-table`

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

#### Comprendre/expliquer chaque ligne

Cette commande nous permet de montrer la table d'adresses mac.

Concrétement ça veut dire que pour communiquer avec l'adresse MAC `0800.274e.1c9f` (par exemple), alors il faut passer par le port `Et0/2`.

### Qu'est-ce que les trames CDP

En effet, il y a des trames CDP qui circulent. Cela signifie Cisco Discovery Protocol (CDP).
C'est un protocole propriétaire de Cisco qui a pour but de fournir des informations sur les voisins connectés.  

```shell script
22    29.215719    aa:bb:cc:00:02:20    CDP/VTP/DTP/PAgP/UDLD    CDP    456    Device ID: IOU2  Port ID: Ethernet0/2 
```

### Déterminer les informations STP 

#### A l'aide des commandes dédiées au protocole

##### Commande `show spanning-tree`

```shell script
IOU3#show spanning-tree

VLAN0001
  Spanning tree enabled protocol rstp
  Root ID    Priority    32769
             Address     aabb.cc00.0100
             Cost        100
             Port        3 (Ethernet0/2)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0300
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    Shr 
Et0/1               Altn BLK 100       128.2    Shr 
Et0/2               Root FWD 100       128.3    Shr 
Et0/3               Desg FWD 100       128.4    Shr 
Et1/0               Desg FWD 100       128.5    Shr 
Et1/1               Desg FWD 100       128.6    Shr 
Et1/2               Desg FWD 100       128.7    Shr 
Et1/3               Desg FWD 100       128.8    Shr 
Et2/0               Desg FWD 100       128.9    Shr 
Et2/1               Desg FWD 100       128.10   Shr 
Et2/2               Desg FWD 100       128.11   Shr 
Et2/3               Desg FWD 100       128.12   Shr 
Et3/0               Desg FWD 100       128.13   Shr 
Et3/1               Desg FWD 100       128.14   Shr 
Et3/2               Desg FWD 100       128.15   Shr 
Et3/3               Desg FWD 100       128.16   Shr 
```

##### Commande `show spanning-tree bridge`

```shell script
IOU3#show spanning-tree bridge

                                                   Hello  Max  Fwd
Vlan                         Bridge ID              Time  Age  Dly  Protocol
---------------- --------------------------------- -----  ---  ---  --------
VLAN0001         32769 (32768,   1) aabb.cc00.0300    2    20   15  rstp 
```

##### Commande `show spanning-tree summary`

```shell script
IOU3#show spanning-tree summary
Switch is in rapid-pvst mode
Root bridge for: none
Extended system ID                      is enabled
Portfast Default                        is disabled
Portfast Edge BPDU Guard Default        is disabled
Portfast Edge BPDU Filter Default       is disabled
Loopguard Default                       is disabled
PVST Simulation Default                 is enabled but inactive in rapid-pvst mode
Bridge Assurance                        is enabled
EtherChannel misconfig guard            is enabled
Configured Pathcost method used is short
UplinkFast                              is disabled
BackboneFast                            is disabled

Name                   Blocking Listening Learning Forwarding STP Active
---------------------- -------- --------- -------- ---------- ----------
VLAN0001                     1         0        0         15         16
---------------------- -------- --------- -------- ---------- ----------
1 vlan                       1         0        0         15         16
```

##### Qui est le root bridge

Le root bridge est `VLAN0001` avec l'adresse MAC `aabb.cc00.0300`.

##### Quels sont les ports désactivés

