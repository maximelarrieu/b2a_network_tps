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

* En se rendant dans le dossier des baux DHCP stockés `$ cd /var/lib/NetworkManager`, on peut afficher le dernier baux crée avec `[admin@localhost lib]$ cat internal-1de272d3-1d0c-422d-b60d-c017de41c887-enp0s3.lease` on obtient alors: 

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

<details open>
<summary>`dig www.reddit.com`</summary>

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

* On créer une nouvelle carte réseau dans VirtualBox (sans cocher la case DHCP) avec pour IP `192.168.57.1` et de la même manière que précédemment, on rédige le fichier correspondant :

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

#### 2. Serveur SSH

* Pour modifier configuration on édite le fichier : `sudo vim /etc/ssh/sshd_config` dans lequel on décommente et modifie la ligne `#Port 22` en `Port 2222` 
Pour activer le port on utilise `sudo firewall-cmd --zone=public --add-port=2222/tcp --permanent` et on ferme le précédent : `firewall-cmd --zone=public --remove-port=22/tcp`

