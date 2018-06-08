### Table des matières
* Contexte
* Prérequis
* Configuration 

### Contexte
Dans un soucis de redondance, un "teaming" réseau pour être demandé afin d'assurer une haute disponibilité. Cette technique permet d'associer 2 cartes réseaux à une 3ème de type virutelle.
En cas de panne d'une des 2 cartes réseaux, la carte virtuelle redirigera le flux vers la carte fonctionnelle. 

### Prérequis
Cette configuration concerne un serveur physique avec 2 cartes réseaux de préférence sur 2 cartes distinctes ou 1 carte et un port sur la carte mère.
Il sera aussi nécessaire d'ouvrir une demande au réseau pour configurer sur ces 2 ports un LACP (ou 802.3ad).
Note: ce changement de configuration devra se réaliser via IRMC car il y aura coupure de l'accès réseau.

### Configration
Création du fichier de configuration de la carte virtuelle et y ajouter le contenu ci-dessous:

```
[root@SI5224 ~]# vi /etc/sysconfig/network-scripts/ifcfg-bond0
DEVICE=bond0
NAME=bond0
TYPE=Bond
BONDING_MASTER=yes
DEFROUTE=yes
IPADDR=157.164.xxx.xxx
PREFIX=24
ONBOOT=yes
BOOTPROTO=none
BONDING_OPTS="mode=4 miimon=100"
GATEWAY=157.164.xxx.1

DNS1=157.164.138.33
DNS2=157.164.138.34
DOMAIN="wallonie.be spw.wallonie.be intra.spw.wallonie.be spw.srv.admin spw.mgt.admin valid.wallonie.be spw.valid.wallonie.be intra.spw.valid.wallonie.be test.wallonie.be spw.test.wallonie.be intra.spw.test.wallonie.be"
IPV6INIT=no
```

Modification des fichiers de configuration des 2 cartes réseaux phyisques:
```
[root@SI5224 ~]# vi /etc/sysconfig/network-scripts/ifcfg-xxxx
BOOTPROTO=none
DEVICE=xxxx
ONBOOT=yes
TYPE=Ethernet
MASTER="bond0"
SLAVE=yes
HWADDR="xx:xx:xx:xx:xx:xx"

[root@SI5224 ~]# vi /etc/sysconfig/network-scripts/ifcfg-yyyy
DEVICE=yyyy
BOOTPROTO=none
ONBOOT=yes
TYPE=Ethernet
MASTER="bond0"
SLAVE=yes
HWADDR="yy:yy:yy:yy:yy:yy"
```

Réaliser un ifup de toute les interfaces et redémarrer les services network:
```
ifup bond0
ifup xxxx
ifup yyyy
systemctl restart network
```
Note: un reboot est plus que vivement conseillé car cela assure une relecture de tous les paramêtres annexes. 

Vérification de l'interface virtuelle et des interfaces associées:
```
[root@SI5224 ~]# cat /proc/net/bonding/bond0
Ethernet Channel Bonding Driver: v3.7.1 (April 27, 2011)

Bonding Mode: IEEE 802.3ad Dynamic link aggregation
Transmit Hash Policy: layer2 (0)
MII Status: up
MII Polling Interval (ms): 100
Up Delay (ms): 0
Down Delay (ms): 0

802.3ad info
LACP rate: slow
Min links: 0
Aggregator selection policy (ad_select): stable
System priority: 65535
System MAC address: xx:xx:xx:xx:xx:xx
Active Aggregator Info:
        Aggregator ID: 2
        Number of ports: 1
        Actor Key: 9
        Partner Key: 1
        Partner Mac Address: 00:00:00:00:00:00

Slave Interface: xxxx
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: xx:xx:xx:xx:xx:xx
Slave queue ID: 0
Aggregator ID: 1
Actor Churn State: churned
Partner Churn State: churned
Actor Churned Count: 1
Partner Churned Count: 1
details actor lacp pdu:
    system priority: 65535
    system mac address: xx:xx:xx:xx:xx:xx
    port key: 9
    port priority: 255
    port number: 1
    port state: 69
details partner lacp pdu:
    system priority: 65535
    system mac address: 00:00:00:00:00:00
    oper key: 1
    port priority: 255
    port number: 1
    port state: 1

Slave Interface: yyyy
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: yy:yy:yy:yy:yy:yy
Slave queue ID: 0
Aggregator ID: 2
Actor Churn State: none
Partner Churn State: churned
Actor Churned Count: 0
Partner Churned Count: 1
details actor lacp pdu:
    system priority: 65535
    system mac address: yy:yy:yy:yy:yy:yy
    port key: 9
    port priority: 255
    port number: 2
    port state: 77
details partner lacp pdu:
    system priority: 65535
    system mac address: 00:00:00:00:00:00
    oper key: 1
    port priority: 255
    port number: 1
    port state: 1

```
