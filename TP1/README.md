# TP: Back to basics
### 0. Etapes préliminaires
* La première carte est la carte NAT qu'on l'on met sur notre VM.
* On joint localement la carte réseau `192.168.56.1` dans un réseau hôte privé. C'est grâce à celle-là que je me connecte en SSH.
* On se connecte à la machine `$ ssh admin@192.168.56.101`, on a accès à notre machine depuis notre local
`[admin@localhost ~]$ cat /etc/centos-release
CentOS Linux release 8.0.1905 (Core)`

### 1. Gather informations

* Pour récupérer la liste des cartes réseau, j'utilise la commande `ip a`. J'obtiens alors une liste de trois résultats, avec la carte réseau et l'ip grâce à laquelle je me suis connecté en SSH : `enp0s3`, la carte `enp0s8` et la carte Ethernet `lo`.
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
[admin@localhost lib]$ 
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
* 
