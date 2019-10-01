# TP: Back to basics
### 0. Etapes préliminaires
* La première carte est la carte NAT qu'on l'on met sur notre VM.
* On joint localement la carte réseau `192.168.56.1` dans un réseau hôte privé. C'est grâce à celle-là que je me connecte en SSH.
* On se connecte à la machine virtuelle `$ ssh admin@192.168.56.101`, on a accès à notre VM depuis notre pc.

```
[admin@localhost ~]$ cat /etc/centos-release
CentOS Linux release 8.0.1905 (Core)
```

### I. Gather informations

* Pour récupérer la liste des cartes réseau, j'utilise la commande `ip a`. 
J'obtiens alors une liste de trois résultats, avec la carte réseau et l'ip grâce à laquelle je me suis connecté en SSH : `enp0s3`, la carte `enp0s8` et la carte Ethernet `lo`.

* En se rendant dans le dossier des baux DHCP stockés `$ cd /var/lib/NetworkManager`, on peut afficher le dernier bail crée avec `[admin@localhost lib]$ cat internal-1de272d3-1d0c-422d-b60d-c017de41c887-enp0s3.lease` on obtient alors: 

```
This is private data. Do not parse.
ADDRESS=192.168.56.103
NETMASK=255.255.255.0
SERVER_ADDRESS=192.168.56.100
T1=600
T2=1050
LIFETIME=1200
CLIENTID=0108002723d842
```

* Table de route :

```
[admin@localhost ~]$ ip route
default via 10.0.3.2 dev enp0s8 proto dhcp metric 101 
10.0.3.0/24 dev enp0s8 proto kernel scope link src 10.0.3.15 metric 101 
192.168.56.0/24 dev enp0s3 proto kernel scope link src 192.168.56.103 metric 100
```


La première ligne indique la mise en place de la route par défaut grâce au DHCP.

La deuxième ligne concerne la carte enp0s8 qui est une route vers le réseau Ynov.

La troisième ligne concerne la carte enp0s3 qui est une route vers le réseau privé de notre machine (pour se connecter en SSH par exemple).

* On peut récupérer la liste des ports en écoute sur la machine en éxecutant :


```
[admin@localhost ~]$ ss -tulpn
Netid                 State                   Recv-Q                  Send-Q                                             Local Address:Port                                     Peer Address:Port                  
udp                   UNCONN                  0                       0                                                      127.0.0.1:323                                           0.0.0.0:*                     
udp                   UNCONN                  0                       0                                          192.168.56.103%enp0s3:68                                            0.0.0.0:*                     
udp                   UNCONN                  0                       0                                               10.0.3.15%enp0s8:68                                            0.0.0.0:*                     
udp                   UNCONN                  0                       0                                                          [::1]:323                                              [::]:*                     
tcp                   LISTEN                  0                       128                                                      0.0.0.0:22                                            0.0.0.0:*                     
tcp                   LISTEN                  0                       128                                                         [::]:22                                               [::]:*  
```


On peut voir le ssh qui écoute sur le `port 22` et nos réseaux sont sur le `port 68`.
* Tout d'abord, pour afficher la liste des DNS, on exécute : `cat /etc/resolv.conf`. Après avoir installer le paquet `bind-utils`, nous pouvons récupérer la l'ip associé au domaine de `www.reddit.com` :

<details>
<summary>dig www.reddit.com</summary>

```
[admin@localhost ~]$ dig www.reddit.com

; <<>> DiG 9.11.4-P2-RedHat-9.11.4-17.P2.el8_0.1 <<>> www.reddit.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 48329
;; flags: qr rd ra; QUERY: 1, ANSWER: 5, AUTHORITY: 13, ADDITIONAL: 27

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;www.reddit.com.			IN	A

;; ANSWER SECTION:
www.reddit.com.		79	IN	CNAME	reddit.map.fastly.net.
reddit.map.fastly.net.	29	IN	A	151.101.65.140
reddit.map.fastly.net.	29	IN	A	151.101.129.140
reddit.map.fastly.net.	29	IN	A	151.101.1.140
reddit.map.fastly.net.	29	IN	A	151.101.193.140

;; AUTHORITY SECTION:
net.			153441	IN	NS	l.gtld-servers.net.
net.			153441	IN	NS	f.gtld-servers.net.
net.			153441	IN	NS	g.gtld-servers.net.
[...]
;; ADDITIONAL SECTION:
a.gtld-servers.net.	153429	IN	A	192.5.6.30
a.gtld-servers.net.	153429	IN	AAAA	2001:503:a83e::2:30
m.gtld-servers.net.	153429	IN	A	192.55.83.30
m.gtld-servers.net.	153429	IN	AAAA	2001:501:b1f9::30
j.gtld-servers.net.	153429	IN	A	192.48.79.30
j.gtld-servers.net.	153429	IN	AAAA	2001:502:7094::30
k.gtld-servers.net.	153429	IN	A	192.52.178.30
k.gtld-servers.net.	153429	IN	AAAA	2001:503:d2d::30
f.gtld-servers.net.	153429	IN	A	192.35.51.30
[...]
;; Query time: 175 msec
;; SERVER: 10.33.10.20#53(10.33.10.20)
;; WHEN: Thu Sep 26 09:48:43 EDT 2019
;; MSG SIZE  rcvd: 935
```
</details>

On sait maintenant que l'adresse ip voulue est : `10.33.10.20`

* L'état actuel du firewall se récupère avec : 
```
[admin@localhost ~]$ systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset>
   Active: active (running) since Thu 2019-09-26 08:58:19 EDT; 56min ago
     Docs: man:firewalld(1)
 Main PID: 758 (firewalld)
    Tasks: 3 (limit: 5060)
   Memory: 32.4M
```
* Les interfaces filtrées se trouvent avec :

```
[root@localhost lib]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
```

* Pour savoir quelles ports sont filtrées on utilise:
`$ sudo nft list table filter`
* Nous jouons avec la nouvelle commande `nft` (nftables) grâce à laquelle on manipule le filtrage réseau, voice un exemple :
* 
```
[root@localhost lib]# nft list table  ip filter
table ip filter {
    chain INPUT {
        type filter hook input priority 0; policy accept;
    }

    chain FORWARD {
        type filter hook forward priority 0; policy accept;
    }

    chain OUTPUT {
        type filter hook output priority 0; policy accept;
    }
}
```

ou encore :

```
[root@localhost lib]# nft list tables
table ip filter
table ip6 filter
table bridge filter
table ip security
table ip raw
table ip mangle
table ip nat
table ip6 security
table ip6 raw
table ip6 mangle
table ip6 nat
table bridge nat
table inet firewalld
table ip firewalld
table ip6 firewalld
```

### II. Edit configuration
#### 1. Configuration cartes réseau

* On modifie la configuration de notre carte réseau. On crée un nouveau fichier `ifcfg-enp0s8` dans lequel on rédige :

```
[admin@localhost ~]$ cat /etc/sysconfig/network-scripts/ifcfg-enp0s8
TYPE="Ethernet"
BOOTPROTO="static"
NAME="enp0s8"
DEVICE="enp0s8"
ONBOOT="yes"
IPADDR=192.168.56.103
NETMASK=255.255.255.0
```

Où on a passer l'adresse ip en statique.
(certaines lignes qui ne sont pas indispensables ont été retirées par nos soins)

Pour finaliser l'étape, on exécute la commande : 
- `nmcli c reload`

et

- `nmcli con up enp0s8`

On créer une nouvelle carte réseau dans VirtualBox (sans cocher la case DHCP) avec pour IP `192.168.57.1` et de la même manière que précédemment, on rédige le fichier correspondant :

```
[admin@localhost ~]$ cat /etc/sysconfig/network-scripts/ifcfg-enp0s9
TYPE="Ethernet"
BOOTPROTO="static"
NAME="enp0s9"
DEVICE="enp0s9"
ONBOOT="yes"
IPADDR=192.168.57.103
NETMASK=255.255.255.0
```

##### Résultat :

```
root@localhost ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:1b:fe:ac brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
       valid_lft 86379sec preferred_lft 86379sec
    inet6 fe80::2bc9:839c:4f72:e37e/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:09:8a:91 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.103/24 brd 192.168.56.255 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::79c6:3352:379a:644b/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
4: enp0s9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:6e:60:42 brd ff:ff:ff:ff:ff:ff
    inet 192.168.57.103/24 brd 192.168.57.255 scope global noprefixroute enp0s9
       valid_lft forever preferred_lft forever
    inet6 fe80::5695:bbe:3536:bc4/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```

#### 2. Serveur SSH

Pour modifier configuration on édite le fichier : 
- `sudo vim /etc/ssh/sshd_config` dans lequel on décommente et modifie la ligne `#Port 22` en `Port 2222` 

- Pour activer le nouveau port on utilise `sudo firewall-cmd --zone=public --add-port=2222/tcp --permanent` 

- On ferme le port précédent : `firewall-cmd --zone=public --remove-port=22/tcp`

- On reload le firewall et le tour est joué : `firewall-cmd --reload`

- On installe semanage avec la commande : `yum install policycoreutils-python-utils-2.8-16.1.el8.noarch`

- On execute `semanage port -a -t ssh_port_t -p tcp 2222` 

##### Résultat :

```
➜  b2a_network_tps git:(master) ✗ ssh root@192.168.56.103 -p 2222
root@192.168.56.103's password: 
Last login: Thu Sep 26 16:02:02 2019
[root@localhost ~]# 
```

### III. Routage simple
###### Tableau récapitulatif des IPs
|VM1         |VM2|
|:-----------|:-:|
|192.168.57.2|192.168.58.2|

###### Configuration des VMs
Pour configurer nos VMs, nous devons définir leurs IPs statiques. Nous modifions alors le fichier d'interface :

```
$ sudo nano /etc/sysconfig/network-scripts/ifcfg-enp0s3

BOOTPROTO="static"
NAME="enp0s3"
DEVICE="enp0s3"
ONBOOT="yes"
IPADDR=192.168.57.2
NETMASK=255.255.255.252
```
Nous procédons de la même façon pour la deuxième VM à laquelle nous avons assigné l'ip `192.168.58.2`. Une fois le fichier modifié, on relance l'interface :

```
$ sudo ifdown enp0s3
$ sudo ifup enp0s3
```

###### Configuration du routeur
Une fois le routeur intégré dans notre topologie dans GNS3, nous ouvrons le terminal du routeur afin d'y accèder et de le configurer :

```
R1# conf t
R1(config)# interface FastEthernet0/0
R1(config-if)# ip address 192.168.58.3 255.255.255.252
R1(config-if)# no shut
R1(config-if)# exit
R1(config)# interface FastEthernet0/1
R1(config-if)# ip address 192.168.58.2 255.255.255.252
R1(config-if)# no shut
R1(config-if)# exit
R1(config)# interface FastEthernet1/0
R1(config-if)# ip address 192.168.56.254 255.255.255.0
R1(config-if)# no shut
R1(config-if)# exit
R1(config)# exit
```
Ici, les cartes `FastEthernet0/0` et `0/1` correspondent aux ports par lesquels passent les VMs et la dernière donne accès au NAS.

### IV. Autres applications et métrologie

#### 1. Commandes

`iftop` n'étant pas présent sur la machine virtuelle, nous l'installons avec la commande `yum install iftop`

Après quelques recherches sur internet, `iftop` est un outil de monitoring qui liste les connexions internet.

```
└─bbbbbbbbbbbbbb┴─bbbbbbbbbbbbbb┴─bbbbbbbbbbbbbb┴─bbbbbbbbbbbbbb┴─bbbbbbbbbbbbb─
localhost.localdomain	   => par10s27-in-f206.1e100.ne     0b    267b     67b
                           <=                               0b    602b    150b
localhost.localdomain	   => 192.168.0.254                 0b    206b     51b
                           <=                               0b    330b     82b
localhost.localdomain	   => ip139.ip-5-196-160.eu	    0b     61b     15b
                           <=                               0b     61b     15b
localhost.localdomain	   => server.gigelf.fr              0b     61b     15b
                           <=                               0b     61b     15b
```
(après avoir fait un `curl` vers google.com)

`iftop` peut être utile si l'on veut surveiller ce que se passe sur sa machine afin de monitorer le flux entrant et sortant.

#### 2. Cockpit

On installe cockpit avec la commande donnée : `sudo dnf install -y cockpit`

On démarre cockpit : `sudo systemctl start cockpit`

Pour savoir sur quel port écoute cockpit : 

```
[root@localhost ~]# ss -tupln
Netid State   Recv-Q   Send-Q         Local Address:Port     Peer Address:Port                                                                                  
udp   UNCONN  0        0                  127.0.0.1:323           0.0.0.0:*      users:(("chronyd",pid=757,fd=6))                                               
udp   UNCONN  0        0           10.0.2.15%enp0s3:68            0.0.0.0:*      users:(("NetworkManager",pid=826,fd=18))                                       
udp   UNCONN  0        0                      [::1]:323              [::]:*      users:(("chronyd",pid=757,fd=7))                                               
tcp   LISTEN  0        128                  0.0.0.0:2222          0.0.0.0:*      users:(("sshd",pid=2102,fd=6))                                                 
tcp   LISTEN  0        128                        *:9090                *:*      users:(("cockpit-ws",pid=3028,fd=3),("systemd",pid=1,fd=24))                   
tcp   LISTEN  0        128                     [::]:2222             [::]:*      users:(("sshd",pid=2102,fd=8)) 
```

On remarque qu'il écoute sur le port 9090.

Le port ne semble pas être ouvert dans le firewall :

```
[root@localhost ~]# firewall-cmd --list-ports
2222/tcp
```

On l'ajoute donc : `firewall-cmd --list-ports 9090/tcp`

C'est bien mieux comme ça !

`[root@localhost ~]# firewall-cmd --list-ports 2222/tcp 9090/tcp`

Je me connecte à l'adresse : https://192.168.56.103:9090 avec les identifiants root et root (du premier coup et à l'instinct).

Nous sommes redirigé sur https://192.168.56.103:9090/system

Nous avons exploré un peu tous les onglets et Cockpit semble être un outil pour garder un oeil et la gérer (on peut reboot la machine par exemple) sur sa machine.

La partie réseau affiche les interfaces. On a un résumé des ports du parefeu.
Nous pouvons voir l'envoi et la réception internet et nous avons accès au journal du réseau.
