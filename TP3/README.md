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

Pour ce qui est de prévoir une augmentation légère du nombre d'adresses IPs disponibles, nous allons prévoir une augmentation de 20% dans notre sous-réseau.

Actuellement notre topologie posséde 38 machines, si l'on respecte l'augmentation de 20%, nous devons donner à notre sous réseau la possibilité d'avoir au moins 46 adresses IPs.

Un sous réseau en /26 nous offre 64 adresses dont 62 disponibles.

##### permettre un accès internet à tout le monde

Sur GNS, nous ajoutons un nuage NAT relié à notre routeur afin de partager internet avec tout le monde.

Exemple de preuve d'accès à internet :

```shell
ADMIN> ping 8.8.8.8
84 bytes from 8.8.8.8 icmp_seq=1 ttl=61 time=144.425 ms
84 bytes from 8.8.8.8 icmp_seq=2 ttl=61 time=106.328 ms
84 bytes from 8.8.8.8 icmp_seq=3 ttl=61 time=73.182 ms
84 bytes from 8.8.8.8 icmp_seq=4 ttl=61 time=845.391 ms
84 bytes from 8.8.8.8 icmp_seq=5 ttl=61 time=97.328 ms
```

#### Pour la partie HARD

##### proposez un nombre de routeur et de switches et précisez à quel endroit physique ils se trouveront

Lors de la mise en place d'une topologie, les frais d'infrastructure doivent êtres pris en compte, ainsi que la sécurité, nous sommes donc parti sur une topologie comportant :

- 1 routeur relié au NAT
- 2 switchs (dont 1 relié à toutes les machines sauf la salle serveurs)

Le routeur et le switch relié à toutes les machines sauf la salle serveur se trouvent dans le bureau principal, à l'abri des autres utilisateurs du réseau. 

Pour ce qui est du dernier routeur, relié à la salle serveur, il se trouve dans la salle serveur qui est sécurisée et fermée.

##### précisez le nombre de câbles nécessaires et une longueur (approximative)

Nous avons 38 machines, donc 38 cables, plus 1 cable entre les 2 switchs et 1 cable entre le routeur et le switch et un cable entre le routeur et le NAT donc : 41 cables.

Si le batiment fait 20 mètres sur 20 mètres :

#### livrer, en plus de l'infra, des éléments qui rendent compte de l'infra (de façon simple)

##### schéma réseau (screen GNS ?)

![Screen gns](https://raw.githubusercontent.com/maximelarrieu/b2a_network_tps/master/TP3/screens/screen-gns.png "Screen GNS de la topologie")

##### référez-vous à la partie I. (tableau des réseaux utilisés, tableau d'adressage)

#### Dans un second temps :

#### BONUS

#### mettre en place les exceptions

##### documentez-vous, proposez des choses

#### mettre en place un serveur DHCP
