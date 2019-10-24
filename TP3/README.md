# TP3 : Routage INTER-VLAN + mise en situation

## I. Router-on-a-stick

### Topologie mise en place

```
             +--+
             |R1|
             +-++
               |
               |                    +---+
               |          +---------+PC4|
+---+        +-+-+      +---+       +---+
|PC1+--------+SW1+------+SW2|
+---+        +-+-+      +-+--+
               |          |  |
               |          |  +------+--+
               |          |         |P1|
             +-+-+      +-+-+       +--+
             |PC2|      |PC3|
             +---+      +---+
```

### Prove me that your setup is actually working

#### think about VLANs, ping, etc.

Ips attribuées aux machines :

Machine | VLAN | IP `net1` | IP `net2` | IP `net3` |  IP `netP`
--- | --- | --- | --- | --- | ---
PC1 | 10 | `10.3.10.1/24` | x | x | x
PC2 | 20 | x | `10.3.20.2/24` | x | x | x
PC3 | 20 | x | `10.3.20.3/24` | x | x | x
PC4 | 30 | x | x |  `10.3.30.4/24` | x | x
P1 | 40 | x | x | x | `10.3.40.1/24` 
R1 | x |  `10.3.10.254/24` | `10.3.20.254/24` | `10.3.30.254/24` | `10.3.40.254/24` 

<details>
<summary>Adresse IP de PC1</summary>

```
PC1> show ip

NAME        : PC1[1]
IP/MASK     : 10.3.10.1/24
GATEWAY     : 10.3.10.254
DNS         : 
MAC         : 00:50:79:66:68:00
LPORT       : 10016
RHOST:PORT  : 127.0.0.1:10017
MTU:        : 1500
```
</details>

<details>
<summary>Adresse IP de PC2</summary>

```
PC2> show ip

NAME        : PC2[1]
IP/MASK     : 10.3.20.2/24
GATEWAY     : 10.3.20.254
DNS         : 
MAC         : 00:50:79:66:68:01
LPORT       : 10018
RHOST:PORT  : 127.0.0.1:10019
MTU:        : 1500
```
</details>

<details>
<summary>Adresse IP de PC3</summary>

```
PC3> show ip

NAME        : PC3[1]
IP/MASK     : 10.3.20.3/24
GATEWAY     : 10.3.20.254
DNS         : 
MAC         : 00:50:79:66:68:02
LPORT       : 10020
RHOST:PORT  : 127.0.0.1:10021
MTU:        : 1500
```
</details>

<details>
<summary>Adresse IP de PC4</summary>

```
PC4> show ip

NAME        : PC4[1]
IP/MASK     : 10.3.30.4/24
GATEWAY     : 10.3.30.254
DNS         : 
MAC         : 00:50:79:66:68:04
LPORT       : 10024
RHOST:PORT  : 127.0.0.1:10025
MTU:        : 1500
```
</details>

<details>
<summary>Adresse IP de P1</summary>

```
P> show ip

NAME        : P[1]
IP/MASK     : 10.3.40.1/24
GATEWAY     : 10.3.40.254
DNS         : 
MAC         : 00:50:79:66:68:03
LPORT       : 10022
RHOST:PORT  : 127.0.0.1:10023
MTU:        : 1500
```
</details>


Pour te prouver, on va tester quelques pings en corrélation avec le tableau :

✅ = peuvent se joindre
❌ = ne peuvent pas se joindre

Réseaux | `net1` |  `net2` |  `net3` |  `netP`
--- | --- | --- | --- | ---
 `net1` | ✅ | ❌ | ❌ | ❌
 `net2` | ❌ | ✅ | ✅ | ✅
 `net3` | ❌ | ✅ | ✅ | ✅
 `netP` | ❌ | ✅ | ✅ | ✅

<details>
<summary>Ping net1 vers net1</summary>

```
PC1> ping 10.3.10.1
10.3.10.1 icmp_seq=1 ttl=64 time=0.001 ms
10.3.10.1 icmp_seq=2 ttl=64 time=0.001 ms
10.3.10.1 icmp_seq=3 ttl=64 time=0.001 ms
10.3.10.1 icmp_seq=4 ttl=64 time=0.001 ms
10.3.10.1 icmp_seq=5 ttl=64 time=0.001 ms
```
</details>

<details>
<summary>Ping net1 vers net2</summary>

```
PC1> ping 10.3.20.2
host (10.3.10.254) not reachable
```
</details>

<details>
<summary>Ping net2 vers net3</summary>

```
PC3> ping 10.3.30.4
84 bytes from 10.3.30.4 icmp_seq=2 ttl=63 time=14.224 ms
84 bytes from 10.3.30.4 icmp_seq=3 ttl=63 time=13.498 ms
84 bytes from 10.3.30.4 icmp_seq=4 ttl=63 time=16.393 ms
84 bytes from 10.3.30.4 icmp_seq=5 ttl=63 time=16.661 ms
```
</details>

<details>
<summary>Ping net3 vers netP</summary>

```
PC4> ping 10.3.40.1
10.3.40.1 icmp_seq=1 timeout
84 bytes from 10.3.40.1 icmp_seq=2 ttl=63 time=13.800 ms
84 bytes from 10.3.40.1 icmp_seq=3 ttl=63 time=15.288 ms
84 bytes from 10.3.40.1 icmp_seq=4 ttl=63 time=19.249 ms
84 bytes from 10.3.40.1 icmp_seq=5 ttl=63 time=16.445 ms
```
</details>

<details>
<summary>Ping netP vers net1</summary>

```
P> ping 10.3.10.1
10.3.10.1 icmp_seq=1 timeout
10.3.10.1 icmp_seq=2 timeout
10.3.10.1 icmp_seq=3 timeout
^C
```
</details>

## II. Cas concret

### TODO

#### Pour la partie SOFT

##### dimensionnez intelligemment les réseaux

###### prévoyez une augmentation légère

##### permettre un accès internet à tout le monde

#### Pour la partie HARD

##### proposez un nombre de routeur et de switches et précisez à quel endroit physique ils se trouveront

##### précisez le nombre de câbles nécessaires et une longueur (approximative)

#### livrer, en plus de l'infra, des éléments qui rendent compte de l'infra (de façon simple)

##### schéma réseau (screen GNS ?)

##### référez-vous à la partie I. (tableau des réseaux utilisés, tableau d'adressage)

#### Dans un second temps :

### BONUS

#### mettre en place les exceptions

##### documentez-vous, proposez des choses

#### mettre en place un serveur DHCP
