# TP3 INFRA : Premiers pas GNS, Cisco et VLAN

On va utiliser GNS3 dans ce TP pour se rapprocher d'un cas réel. On va focus sur l'aspect switching, avec du matériel Cisco. On va aussi mettre en place des VLANs.


# Sommaire

- [TP3 INFRA : Premiers pas GNS, Cisco et VLAN](#tp3-infra--premiers-pas-gns-cisco-et-vlan)
- [Sommaire](#sommaire)
- [0. Prérequis](#0-prérequis)
- [I. Dumb switch](#i-dumb-switch)
  - [1. Topologie 1](#1-topologie-1)
  - [2. Adressage topologie 1](#2-adressage-topologie-1)
  - [3. Setup topologie 1](#3-setup-topologie-1)
- [II. VLAN](#ii-vlan)
  - [1. Topologie 2](#1-topologie-2)
  - [2. Adressage topologie 2](#2-adressage-topologie-2)
    - [3. Setup topologie 2](#3-setup-topologie-2)
- [III. Ptite VM DHCP](#iii-ptite-vm-dhcp)

# 0. Prérequis

➜ **GNS3 installé et prêt à l'emploi**

- [GNS3VM](https://www.gns3.com/software/download-vm) fonctionnelle
- de quoi faire tourner un switch Cisco
  - [IOU2 L2 dispo ici](http://dl.nextadmin.net/dl/EVE-NG-image/iol/bin/i86bi_linux_l2-adventerprisek9-ms.SSA.high_iron_20180510.bin)
- de quoi faire tourner un routeur Cisco
  - [image d'un 3745 dispo ici](http://dl.nextadmin.net/dl/EVE-NG-image/dynamips/c3725-adventerprisek9-mz.124-15.T14.image)


# I. Dumb switch

## 1. Topologie 1


## 2. Adressage topologie 1

| Node  | IP            |
| ----- | ------------- |
| `pc1` | `10.3.1.1/24` |
| `pc2` | `10.3.1.2/24` |

## 3. Setup topologie 1

🌞 **Commençons simple**

- définissez les IPs statiques sur les deux VPCS

```bash
PC1> show ip

NAME        : PC1[1]
IP/MASK     : 10.3.1.1/24
GATEWAY     : 0.0.0.0
DNS         :
MAC         : 00:50:79:66:68:00
LPORT       : 20004
RHOST:PORT  : 127.0.0.1:20005
MTU         : 1500

```
```bash
PC2> show ip

NAME        : PC2[1]
IP/MASK     : 10.3.1.2/24
GATEWAY     : 0.0.0.0
DNS         :
MAC         : 00:50:79:66:68:01
LPORT       : 20006
RHOST:PORT  : 127.0.0.1:20007
MTU         : 1500

```
- `ping` un VPCS depuis l'autre

```bash
PC1> ping 10.3.1.2

84 bytes from 10.3.1.2 icmp_seq=1 ttl=64 time=0.166 ms
84 bytes from 10.3.1.2 icmp_seq=2 ttl=64 time=0.296 ms
```
- afficher la CAM table du switch et vérifier les MAC qui s'y trouvent (Google pour ça, c'est pas dans le mémo) 
```bash

IOU1#show mac address-table
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    0800.27fc.5e8a    DYNAMIC     Et1/1
Total Mac Addresses for this criterion: 1

```

# II. VLAN

On va rajouter **un troisième client** qui, bien que dans le même réseau, sera **isolé des autres grâce aux *VLANs***.


Le principe est simple :

- déclaration du VLAN sur tous les switches
  - un VLAN a forcément un ID (un entier)
  - bonne pratique, on lui met un nom
- sur chaque switch, on définit le VLAN associé à chaque port
  - genre "sur le port 35, c'est un client du VLAN 20 qui est branché"

## 1. Topologie 2

## 2. Adressage topologie 2

| Node  | IP            | VLAN |
| ----- | ------------- | ---- |
| `pc1` | `10.3.1.1/24` | 10   |
| `pc2` | `10.3.1.2/24` | 10   |
| `pc3` | `10.3.1.3/24` | 20   |

### 3. Setup topologie 2

🌞 **Adressage**

- définissez les IPs statiques sur tous les VPCS
```bash
PC1> show ip

NAME        : PC1[1]
IP/MASK     : 10.3.1.1/24
GATEWAY     : 0.0.0.0
DNS         :
MAC         : 00:50:79:66:68:00
LPORT       : 20004
RHOST:PORT  : 127.0.0.1:20005
MTU         : 1500

```
```bash
PC2> show ip

NAME        : PC2[1]
IP/MASK     : 10.3.1.2/24
GATEWAY     : 0.0.0.0
DNS         :
MAC         : 00:50:79:66:68:01
LPORT       : 20006
RHOST:PORT  : 127.0.0.1:20007
MTU         : 1500

```
```bash
PC3> show ip

NAME        : PC3[1]
IP/MASK     : 10.3.1.3/24
GATEWAY     : 0.0.0.0
DNS         :
MAC         : 00:50:79:66:68:02
LPORT       : 20010
RHOST:PORT  : 127.0.0.1:20011
MTU         : 1500

```
- vérifiez avec des `ping` que tout le monde se ping
```bash

PC1> ping 10.3.1.2

84 bytes from 10.3.1.2 icmp_seq=1 ttl=64 time=0.122 ms
84 bytes from 10.3.1.2 icmp_seq=2 ttl=64 time=0.362 ms

PC1> ping 10.3.1.3

84 bytes from 10.3.1.3 icmp_seq=1 ttl=64 time=0.182 ms
84 bytes from 10.3.1.3 icmp_seq=2 ttl=64 time=0.354 ms

```
```bash
PC2> ping 10.3.1.1

84 bytes from 10.3.1.1 icmp_seq=1 ttl=64 time=0.287 ms
84 bytes from 10.3.1.1 icmp_seq=2 ttl=64 time=0.209 ms

PC2> ping 10.3.1.3

84 bytes from 10.3.1.3 icmp_seq=1 ttl=64 time=0.270 ms
```
```bash

PC3> ping 10.3.1.1

84 bytes from 10.3.1.1 icmp_seq=1 ttl=64 time=0.245 ms
84 bytes from 10.3.1.1 icmp_seq=2 ttl=64 time=0.246 ms

PC3> ping 10.3.1.2

84 bytes from 10.3.1.2 icmp_seq=1 ttl=64 time=0.249 ms
84 bytes from 10.3.1.2 icmp_seq=2 ttl=64 time=0.261 ms

```

🌞 **Configuration des VLANs**

-  référez-vous [à la section VLAN du mémo Cisco](../../cours/memo/memo_cisco.md#8-vlan)

- déclaration des VLANs sur le switch `sw1`
```bash
IOU1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
IOU1(config)#vlan 10
IOU1(config-vlan)#name vlan_10
IOU1(config-vlan)#exit
IOU1(config)#vlan 20
IOU1(config-vlan)#name vlan_20
IOU1(config-vlan)#exit
IOU1(config)#exit
IOU1#
*Oct 26 09:38:07.873: %SYS-5-CONFIG_I: Configured from console by console
IOU1#show vlan

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et0/0, Et0/1, Et0/2, Et0/3
                                                Et1/0, Et1/1, Et1/2, Et1/3
                                                Et2/0, Et2/1, Et2/2, Et2/3
                                                Et3/0, Et3/1, Et3/2, Et3/3
10   vlan_10                          active
20   vlan_20                          active
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup

VLAN Type  SAID       MTU   Parent RingNo BridgeNo Stp  BrdgMode Trans1 Trans2
---- ----- ---------- ----- ------ ------ -------- ---- -------- ------ ------
1    enet  100001     1500  -      -      -        -    -        0      0
10   enet  100010     1500  -      -      -        -    -        0      0
20   enet  100020     1500  -      -      -        -    -        0      0
1002 fddi  101002     1500  -      -      -        -    -        0      0
1003 tr    101003     1500  -      -      -        -    -        0      0
1004 fdnet 101004     1500  -      -      -        ieee -        0      0
1005 trnet 101005     1500  -      -      -        ibm  -        0      0
 --More--

```
- ajout des ports du switches dans le bon VLAN (voir [le tableau d'adressage de la topo 2 juste au dessus](#2-adressage-topologie-2))
  - ici, tous les ports sont en mode *access* : ils pointent vers des clients du réseau

```bash
IOU1(config)#interface Ethernet0/0
IOU1(config-if)#switchport mode access
IOU1(config-if)#switchport access vlan 10
```
```bash
IOU1(config)#interface Ethernet0/1
IOU1(config-if)#switchport mode access
IOU1(config-if)#switchport access vlan 10
```
```bash
IOU1(config)#interface Ethernet0/2
IOU1(config-if)#switchport mode access
IOU1(config-if)#switchport access vlan 20
```
```bash
IOU1#show vlan br

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et0/3, Et1/0, Et1/1, Et1/2
                                                Et1/3, Et2/0, Et2/1, Et2/2
                                                Et2/3, Et3/0, Et3/1, Et3/2
                                                Et3/3
10   vlan_10                          active    Et0/0, Et0/1
20   vlan_20                          active    Et0/2
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup
```
🌞 **Vérif**

- `pc1` et `pc2` doivent toujours pouvoir se ping
```bash
PC1> ping 10.3.1.2

84 bytes from 10.3.1.2 icmp_seq=1 ttl=64 time=0.199 ms
84 bytes from 10.3.1.2 icmp_seq=2 ttl=64 time=0.260 ms
```
```bash
PC2> ping 10.3.1.1

84 bytes from 10.3.1.1 icmp_seq=1 ttl=64 time=0.295 ms
84 bytes from 10.3.1.1 icmp_seq=2 ttl=64 time=0.269 ms
```
- `pc3` ne ping plus personne
```bash
PC3> ping 10.3.1.1

host (10.3.1.1) not reachable

PC3> ping 10.3.1.2

host (10.3.1.2) not reachable
```

# III. Ptite VM DHCP

On va ajouter une VM dans la topologie, histoire que vous voyiez cette aspect de GNS.

| Node          | IP              | VLAN |
| ------------- | --------------- | ---- |
| `pc1`         | `10.3.1.1/24`   | 10   |
| `pc2`         | `10.3.1.2/24`   | 10   |
| `pc3`         | `10.3.1.3/24`   | 20   |
| `pc4`         | X               | 20   |
| `pc5`         | X               | 10   |
| `dhcp.tp3.b2` | `10.3.1.253/24` | 20   |

```bash

IOU1#show vlan br

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et1/2, Et1/3, Et2/0, Et2/1
                                                Et2/2, Et2/3, Et3/0, Et3/1
                                                Et3/2, Et3/3
10   vlan_10                          active    Et0/0, Et0/1, Et1/0
20   vlan_20                          active    Et0/2, Et0/3, Et1/1
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup
```

🌞 **VM `dhcp.tp3.b2`**

- Rocky Linux, IP statique, nom défini à `dhcp.tp3.b2`, SELinux désactivé, firewall activé, système à jour, NORMAL LA ROUTINE QUOI
- installez un serveur DHCP
  - il doit délivrer des IPs entre `10.3.1.100` et `10.3.1.200`
- vérifier avec le `pc4` que vous pouvez récupérer une IP en DHCP

```bash
PC4> ip dhcp
DDORA IP 10.3.1.100/24 GW 10.3.1.1
```
- vérifier avec le `pc5` que vous ne pouvez PAS récupérer une IP en DHCP

```bash
PC5> ip dhcp
DDD
Can't find dhcp server
```
