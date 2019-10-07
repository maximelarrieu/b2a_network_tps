# TP2 : Network low-level, Switching

## I. Simplest setup

Topologie mise en place :

```
+-----+        +-------+        +-----+
| PC1 +--------+  SW1  +--------+ PC2 |
+-----+        +-------+        +-----+
```

Ping de PC1 vers PC2 :



Le protocole utilisé est ICMP qui signifie Internet Control Message Protocol.

Ce switch ne possède pas d'IP car sa mission est de faire le pont entre le PC1 et le PC2, il n'est qu'un intermédiaire.

Les machines ont besoin d'une IP pour pouvoir se ping et plus généralement pour communiquer car c'est le protocole pour 
discuter à travers le réseau qui est "portée" par une carte réseau. (cf mémo)