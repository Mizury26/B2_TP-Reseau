# TP6 : STP, OSPF, bigger infra

TP où on avance sur des protocoles toujours très utilisés, mais qu'on va trouver surtout dans des gros réseaux.

On considère que ça reste des bases en admin réseau. Au menu donc :

- expérimenter STP
  - ptite topo simple avec plusieurs switches
- expérimenter OSPF
  - routage dynamique : les routeurs se partagent leurs routes
  - topo un peu plus fat
  - on en profite pour voir du DHCP Relay

> *Dans ce TP, pas de VLANs pour simplifier, et focus sur le sujet. Dans la vie réelle, les VLANs sont omniprésents.*


## Sommaire

- [TP6 : STP, OSPF, bigger infra](#tp6--stp-ospf-bigger-infra)
  - [Sommaire](#sommaire)
  - [0. Setup](#0-setup)
  - [I. STP](#i-stp)
  - [II. OSPF](#ii-ospf)
  - [III. DHCP relay](#iii-dhcp-relay)
  - [IV. Bonus](#iv-bonus)
    - [1. ACL](#1-acl)

## 0. Setup

➜ **Faites chauffer GNS**

- VM Rocky prête à être clonée
- du switch et routeur Cisco, on reste sur les images des précédents TPs

Hop petit hint en passant, on peut configurer le hostname des routeurs Cisco avec :

```cisco
R1# conf t
R1(config)# hostname meow
meow(config)# incroyable
```

➜ **Augmentez la RAM des routeurs** dans GNS, surtout celui qui est connecté à internet (partie II.) à 256M

## I. STP

On va setup STP, au sein d'une topo simple pour que vous le voyiez en action.

![Topo STP](./img/topo_stp.png)

🌞 **Configurer STP sur les 3 switches**

- bon c'est surtout activé par défaut en 2023
- je veux bien un `show spanning-tree`
  - y'a forcément un port en état *BLK* là

```bash
IOU1#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.0100
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0100
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    P2p
Et0/1               Desg FWD 100       128.2    P2p
Et0/2               Desg FWD 100       128.3    P2p
```
```bash
IOU2#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.0100
             Cost        100
             Port        1 (Ethernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0200
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Root FWD 100       128.1    P2p
Et0/1               Desg FWD 100       128.2    P2p
Et0/2               Desg FWD 100       128.3    P2p
```
```bash
IOU3#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.0100
             Cost        100
             Port        1 (Ethernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0300
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Root FWD 100       128.1    P2p
Et0/1               Altn BLK 100       128.2    P2p
Et0/2               Desg FWD 100       128.3    P2p
```

🌞 **Altérer le spanning-tree**

- désactiver juste un port de un switch pour provoquer la mise à jour de STP

```bash
IOU1#conf t
IOU1(config)#interface Ethernet0/0
IOU1(config-if)#shutdown
IOU1(config-if)#end
*Dec  1 09:33:38.897: %LINK-5-CHANGED: Interface Ethernet0/0, changed state to administratively down
*Dec  1 09:33:39.904: %LINEPROTO-5-UPDOWN: Line protocol on Interface Ethernet0/0, changed state to down
```
- `show spanning-tree` pour voir la diff
```bash
IOU1#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.0100
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0100
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  15  sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Desg FWD 100       128.2    P2p
Et0/2               Desg FWD 100       128.3    P2p
```
```bash
IOU2#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.0100
             Cost        200
             Port        2 (Ethernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0200
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    P2p
Et0/1               Root FWD 100       128.2    P2p
Et0/2               Desg FWD 100       128.3    P2p
```
```bash
IOU3#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.0100
             Cost        100
             Port        1 (Ethernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0300
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Root FWD 100       128.1    P2p
Et0/1               Altn BLK 100       128.2    P2p
Et0/2               Desg FWD 100       128.3    P2p
```

> **Ré-activer ce port après la manip** inutile de le laisser désactivé.

```bash
IOU1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
IOU1(config)#interface Ethernet0/0
IOU1(config-if)#no shutdown
IOU1(config-if)#
*Dec  1 10:06:51.902: %LINK-3-UPDOWN: Interface Ethernet0/0, changed state to up
*Dec  1 10:06:52.908: %LINEPROTO-5-UPDOWN: Line protocol on Interface Ethernet0/0, changed state to up
```
🌞 **Altérer le spanning-tree** en modifiant le coût d'un lien

- modifier le coût d'un lien existant pour modifier l'arbre spanning-tree

```bash
IOU2(config)#interface Ethernet0/0
IOU2(config-if)#spanning-tree vlan 1 port-priority 16
```
```bash

IOU1#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.0100
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0100
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    P2p
Et0/1               Desg FWD 100       128.2    P2p
Et0/2               Desg FWD 100       128.3    P2p
```
```bash

IOU2#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.0100
             Cost        100
             Port        1 (Ethernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0200
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Root FWD 100        16.1    P2p
Et0/1               Desg FWD 100       128.2    P2p
Et0/2               Desg FWD 100       128.3    P2p
```
```bash
IOU3#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.0100
             Cost        100
             Port        1 (Ethernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0300
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Root FWD 100       128.1    P2p
Et0/1               Altn BLK 100       128.2    P2p
Et0/2               Desg FWD 100       128.3    P2p
```

🦈 **`tp6_stp.pcapng`**

- capturez du trafic STP, quelques trames
- interprétez les trames (rien dans le rendu à écrire, mais juste, fais l'effort de capter ce que les switches échangent comme message)

[capture STP](./capture/tp6_stp.pcapng)

## II. OSPF

OSPF donc, routage dynamique.

On va se cantonner à le setup de façon simple, et ensuite on mettra en place un service qui consomme ce routage en partie III.

![Topo OSPF](./img/topo_ospf.png)

> Ce sont les *areas* OSPF qui sont représentées en couleur, pas des réseaux. 🌸

➜ Tableau d'adressage

- la logique de l'adressage que je vous propose :
  - choix de masque
    - du `/24` pour les réseaux où y'a des clients
      - classique, simple
    - du `/30` pour les réseaux entre les routeurs
      - comme ça, on permet vraiment explicitement que deux IPs sur ces réseaux
  - choix des octets
    - `10.6.` pour les deux premiers octets
      - 10 pas chiant comme d'hab
      - 6 pour TP6 comme d'hab
    - pour le troisième octet
      - entre les routeurs : `10.6.13.` l'octet qui suit :
        - 13 indique le réseau entre le routeur 1 et le routeur 3
        - 13 et pas 31 parce que je lis de gauche à droite perso
      - réseaux clients : `10.6.1.`
        - arbitraire, y'a un réseau 1, un réseau 2, etc.

| Node          | `10.6.1.0/24` | `10.6.2.0/24` | `10.6.3.0/24` | `10.6.41.0/30` | `10.6.13.0/30` | `10.6.21.0/30` | `10.6.23.0/30` | `10.6.52.0/30` |
| ------------- | ------------- | ------------- | ------------- | -------------- | -------------- | -------------- | -------------- | -------------- |
| `waf.tp6.b1`  | `10.6.1.11`   | ❌            | ❌            | ❌             | ❌             | ❌             | ❌             | ❌             |
| `dhcp.tp6.b1` | `10.6.1.253`  | ❌            | ❌            | ❌             | ❌             | ❌             | ❌             | ❌             |
| `meo.tp6.b1`  | ❌            | `10.6.2.11`   | ❌            | ❌             | ❌             | ❌             | ❌             | ❌             |
| `john.tp6.b1` | ❌            | ❌            | `10.6.3.11`   | ❌             | ❌             | ❌             | ❌             | ❌             |
| `R1`          | ❌            | ❌            | `10.6.3.254`  | `10.6.41.1`    | `10.6.13.1`    | `10.6.21.1`    | ❌             | ❌             |
| `R2`          | ❌            | ❌            | ❌            | ❌             | ❌             | `10.6.21.2`    | `10.6.23.2`    | `10.6.52.2`    |
| `R3`          | ❌            | ❌            | ❌            | ❌             | `10.6.13.2`    | ❌             | `10.6.23.1`    | ❌             |
| `R4`          | `10.6.1.254`  | `10.6.2.254`  | ❌            | `10.6.41.2`    | ❌             | ❌             | ❌             | ❌             |
| `R5`          | ❌            | ❌            | ❌            | ❌             | ❌             | ❌             | ❌             | `10.6.52.1`    |

🌞 **Montez la topologie**

- IP statiques sur tout le monde
  - assurez-vous que les pings passent au sein de chacun des LANs
  - au fur et à mesure que vous configurez
- configuration d'un NAT sur le routeur connecté à internet
- **aucune route statique ne doit être ajoutée nulle part**
- définissez aux clients (VPCS ou VMs) des IPs statiques et définissez leur gateway
  - ils auront toujours pas internet, car leur routeur n'a pas internet !
- aucune configuration particulière à faire sur `dhcp.tp6.b2` pour le moment, on fera ça en partie III.
  - juste une IP statique, pas de setup particulier

➜ Bref...

- IP statiques partout
- clients avec une gateway définie
- et un NAT sur le routeur de bordurec

```bash
waf.tp6.b1> sh ip

NAME        : waf.tp6.b1[1]
IP/MASK     : 10.6.1.11/24
GATEWAY     : 10.6.1.254
DNS         : 1.1.1.1
MAC         : 00:50:79:66:68:02
LPORT       : 20004
RHOST:PORT  : 127.0.0.1:20005
MTU         : 1500

dhcp.tp6.b1> sh ip

NAME        : dhcp.tp6.b1[1]
IP/MASK     : 10.6.1.253/24
GATEWAY     : 10.6.1.254
DNS         : 1.1.1.1
MAC         : 00:50:79:66:68:03
LPORT       : 20002
RHOST:PORT  : 127.0.0.1:20003
MTU         : 1500

meo.tp6.b1> sh ip

NAME        : meo.tp6.b1[1]
IP/MASK     : 10.6.2.11/24
GATEWAY     : 10.6.2.254
DNS         : 1.1.1.1
MAC         : 00:50:79:66:68:00
LPORT       : 20006
RHOST:PORT  : 127.0.0.1:20007
MTU         : 1500

john.tp6.b1> sh ip

NAME        : john.tp6.b1[1]
IP/MASK     : 10.6.3.11/24
GATEWAY     : 10.6.3.254
DNS         : 1.1.1.1
MAC         : 00:50:79:66:68:01
LPORT       : 20008
RHOST:PORT  : 127.0.0.1:20009
MTU         : 1500

R1#sh ip int br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            10.6.21.1       YES manual up                    up
FastEthernet0/1            10.6.13.1       YES manual up                    up
FastEthernet1/0            10.6.41.1       YES manual up                    up
FastEthernet2/0            10.6.3.254      YES manual up                    up

R2#sh ip int br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            10.6.21.2       YES manual up                    up
FastEthernet0/1            10.6.23.2       YES manual up                    up
FastEthernet1/0            10.6.52.2       YES manual up                    up
FastEthernet2/0            unassigned      YES unset  up                    up

R3#sh ip int br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            10.6.13.2       YES manual up                    up
FastEthernet0/1            10.6.23.1       YES manual up                    up
FastEthernet1/0            unassigned      YES unset  up                    up
FastEthernet2/0            unassigned      YES unset  up                    up

R4#sh ip int br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            10.6.41.2       YES manual up                    up
FastEthernet0/1            10.6.1.254      YES manual up                    up
FastEthernet1/0            10.6.2.254      YES manual up                    up
FastEthernet2/0            unassigned      YES unset  up                    up

R5#sh ip int br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            192.168.122.93  YES DHCP   up                    up
FastEthernet0/1            10.6.52.1       YES manual up                    up
FastEthernet1/0            unassigned      YES unset  up                    up
FastEthernet2/0            unassigned      YES unset  up                    up

R5#ping 1.1.1.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 20/28/40 ms
```

On va pas ajouter toutes les routes sur tous les routeurs, ce serait une giga plaie à faire, à maintenir, et peu résilient.

OSPF donc.

🌞 **Configurer OSPF sur tous les routeurs**

- tous les routeurs doivent partager tous les réseaux auxquels ils sont connectés
- un petit `show running-config` où vous enlevez ce que vous n'avez pas tapé pour le rendu !
- et un `show ip ospf neighbor` + `show ip route` sur chaque routeur
- n'oubliez pas de partager la route par défaut de R5 avec une commande OSPF spécifique [voir mémo](../../../cours/memo/cisco.md)

```bash
R5(config)#router ospf 1
R5(config-router)#default-information originate always
R5(config-router)#exit
R5(config)#exit
```

```bash
R1#show running-config
Building configuration...
interface FastEthernet0/0
 ip address 10.6.21.1 255.255.255.252
 duplex auto
 speed auto
!
interface FastEthernet0/1
 ip address 10.6.13.1 255.255.255.252
 duplex auto
 speed auto
!
interface FastEthernet1/0
 ip address 10.6.41.1 255.255.255.252
 duplex auto
 speed auto
!
interface FastEthernet2/0
 ip address 10.6.3.254 255.255.255.0
 duplex auto
 speed auto
!
router ospf 1
 router-id 1.1.1.1
 log-adjacency-changes
 network 10.6.3.0 0.0.0.255 area 2
 network 10.6.13.0 0.0.0.3 area 0
 network 10.6.21.0 0.0.0.3 area 0
 network 10.6.41.0 0.0.0.3 area 3

R1#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           1   FULL/BDR        00:00:35    10.6.21.2       FastEthernet0/0
3.3.3.3           1   FULL/BDR        00:00:32    10.6.13.2       FastEthernet0/1
4.4.4.4           1   FULL/BDR        00:00:39    10.6.41.2       FastEthernet1/0

R1#show ip route
Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route

Gateway of last resort is not set

     10.0.0.0/8 is variably subnetted, 8 subnets, 2 masks
C       10.6.13.0/30 is directly connected, FastEthernet0/1
O       10.6.1.0/24 [110/11] via 10.6.41.2, 00:21:22, FastEthernet1/0
O       10.6.2.0/24 [110/2] via 10.6.41.2, 00:21:22, FastEthernet1/0
C       10.6.3.0/24 is directly connected, FastEthernet2/0
C       10.6.21.0/30 is directly connected, FastEthernet0/0
O       10.6.23.0/30 [110/20] via 10.6.21.2, 00:25:59, FastEthernet0/0
                     [110/20] via 10.6.13.2, 00:23:23, FastEthernet0/1
C       10.6.41.0/30 is directly connected, FastEthernet1/0
O IA    10.6.52.0/30 [110/11] via 10.6.21.2, 00:25:33, FastEthernet0/0
```
```bash
R2#show running-config
Building configuration...

interface FastEthernet0/0
 ip address 10.6.21.2 255.255.255.252
 duplex auto
 speed auto
!
interface FastEthernet0/1
 ip address 10.6.23.2 255.255.255.252
 duplex auto
 speed auto
!
interface FastEthernet1/0
 ip address 10.6.52.2 255.255.255.252
 duplex auto
 speed auto
!
interface FastEthernet2/0
 no ip address
 duplex auto
 speed auto
!
router ospf 1
 router-id 2.2.2.2
 log-adjacency-changes
 network 10.6.21.0 0.0.0.3 area 0
 network 10.6.23.0 0.0.0.3 area 0
 network 10.6.52.0 0.0.0.3 area 1

R2#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
3.3.3.3           1   FULL/BDR        00:00:32    10.6.23.1       FastEthernet0/1
1.1.1.1           1   FULL/DR         00:00:35    10.6.21.1       FastEthernet0/0
5.5.5.5           1   FULL/BDR        00:00:39    10.6.52.1       FastEthernet1/0

R2#show ip route
Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route

Gateway of last resort is not set

     10.0.0.0/8 is variably subnetted, 8 subnets, 2 masks
O       10.6.13.0/30 [110/20] via 10.6.23.1, 00:25:59, FastEthernet0/1
                     [110/20] via 10.6.21.1, 00:28:13, FastEthernet0/0
O IA    10.6.1.0/24 [110/21] via 10.6.21.1, 00:23:58, FastEthernet0/0
O IA    10.6.2.0/24 [110/12] via 10.6.21.1, 00:23:58, FastEthernet0/0
O IA    10.6.3.0/24 [110/11] via 10.6.21.1, 00:28:13, FastEthernet0/0
C       10.6.21.0/30 is directly connected, FastEthernet0/0
C       10.6.23.0/30 is directly connected, FastEthernet0/1
O IA    10.6.41.0/30 [110/11] via 10.6.21.1, 00:28:14, FastEthernet0/0
C       10.6.52.0/30 is directly connected, FastEthernet1/0
```
```bash
R3#show running-config
Building configuration...
interface FastEthernet0/0
 ip address 10.6.13.2 255.255.255.252
 duplex auto
 speed auto
!
interface FastEthernet0/1
 ip address 10.6.23.1 255.255.255.252
 duplex auto
 speed auto
!
interface FastEthernet1/0
 no ip address
 duplex auto
 speed auto
!
interface FastEthernet2/0
 no ip address
 duplex auto
 speed auto
!
router ospf 1
 router-id 3.3.3.3
 log-adjacency-changes
 network 10.6.13.0 0.0.0.3 area 0
 network 10.6.23.0 0.0.0.3 area 0
!

R3#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           1   FULL/DR         00:00:33    10.6.23.2       FastEthernet0/1
1.1.1.1           1   FULL/DR         00:00:31    10.6.13.1       FastEthernet0/0

R3#show ip route
Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route

Gateway of last resort is not set

     10.0.0.0/8 is variably subnetted, 8 subnets, 2 masks
C       10.6.13.0/30 is directly connected, FastEthernet0/0
O IA    10.6.1.0/24 [110/21] via 10.6.13.1, 00:28:41, FastEthernet0/0
O IA    10.6.2.0/24 [110/12] via 10.6.13.1, 00:28:41, FastEthernet0/0
O IA    10.6.3.0/24 [110/11] via 10.6.13.1, 00:31:11, FastEthernet0/0
O       10.6.21.0/30 [110/20] via 10.6.23.2, 00:30:42, FastEthernet0/1
                     [110/20] via 10.6.13.1, 00:31:11, FastEthernet0/0
C       10.6.23.0/30 is directly connected, FastEthernet0/1
O IA    10.6.41.0/30 [110/11] via 10.6.13.1, 00:31:12, FastEthernet0/0
O IA    10.6.52.0/30 [110/11] via 10.6.23.2, 00:30:44, FastEthernet0/1
```
```bash
R4#show running-config
Building configuration...
interface FastEthernet0/0
 ip address 10.6.41.2 255.255.255.252
 duplex auto
 speed auto
!
interface FastEthernet0/1
 ip address 10.6.1.254 255.255.255.0
 duplex auto
 speed auto
!
interface FastEthernet1/0
 ip address 10.6.2.254 255.255.255.0
 duplex auto
 speed auto
!
interface FastEthernet2/0
 no ip address
 duplex auto
 speed auto
!
router ospf 1
 router-id 4.4.4.4
 log-adjacency-changes
 network 10.6.1.0 0.0.0.255 area 3
 network 10.6.2.0 0.0.0.255 area 3
 network 10.6.41.0 0.0.0.3 area 3
!

R4#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1           1   FULL/DR         00:00:32    10.6.41.1       FastEthernet0/0

R4#show ip route
Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route

Gateway of last resort is not set

     10.0.0.0/8 is variably subnetted, 8 subnets, 2 masks
O IA    10.6.13.0/30 [110/20] via 10.6.41.1, 00:31:31, FastEthernet0/0
C       10.6.1.0/24 is directly connected, FastEthernet0/1
C       10.6.2.0/24 is directly connected, FastEthernet1/0
O IA    10.6.3.0/24 [110/11] via 10.6.41.1, 00:31:31, FastEthernet0/0
O IA    10.6.21.0/30 [110/20] via 10.6.41.1, 00:31:31, FastEthernet0/0
O IA    10.6.23.0/30 [110/30] via 10.6.41.1, 00:31:31, FastEthernet0/0
C       10.6.41.0/30 is directly connected, FastEthernet0/0
O IA    10.6.52.0/30 [110/21] via 10.6.41.1, 00:31:32, FastEthernet0/0
```
```bash
R5#show running-config
Building configuration...

interface FastEthernet0/0
 ip address dhcp
 ip nat outside
 ip virtual-reassembly
 duplex auto
 speed auto
!
interface FastEthernet0/1
 ip address 10.6.52.1 255.255.255.252
 ip nat inside
 ip virtual-reassembly
 duplex auto
 speed auto
!
interface FastEthernet1/0
 no ip address
 duplex auto
 speed auto
!
interface FastEthernet2/0
 no ip address
 duplex auto
 speed auto
!
router ospf 1
 router-id 5.5.5.5
 log-adjacency-changes
 network 10.6.52.0 0.0.0.3 area 1
 default-information originate always
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
ip nat inside source list 1 interface FastEthernet0/0 overload
!
access-list 1 permit any

R5#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           1   FULL/DR         00:00:37    10.6.52.2       FastEthernet0/1

R5#show ip route
Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route

Gateway of last resort is 192.168.122.1 to network 0.0.0.0

C    192.168.122.0/24 is directly connected, FastEthernet0/0
     10.0.0.0/8 is variably subnetted, 8 subnets, 2 masks
O IA    10.6.13.0/30 [110/30] via 10.6.52.2, 00:35:27, FastEthernet0/1
O IA    10.6.1.0/24 [110/31] via 10.6.52.2, 00:35:27, FastEthernet0/1
O IA    10.6.2.0/24 [110/22] via 10.6.52.2, 00:35:27, FastEthernet0/1
O IA    10.6.3.0/24 [110/21] via 10.6.52.2, 00:35:27, FastEthernet0/1
O IA    10.6.21.0/30 [110/20] via 10.6.52.2, 00:35:27, FastEthernet0/1
O IA    10.6.23.0/30 [110/20] via 10.6.52.2, 00:35:29, FastEthernet0/1
O IA    10.6.41.0/30 [110/21] via 10.6.52.2, 00:35:29, FastEthernet0/1
C       10.6.52.0/30 is directly connected, FastEthernet0/1
S*   0.0.0.0/0 [254/0] via 192.168.122.1
```

🌞 **Test**

- faites des `ping` dans tous les sens
- c'est simple hein : normalement tout le monde peut ping tout le monde
- et même tout le monde a internet y compris les clients
- mettez moi quelques `ping` dans le compte-rendu

```bash
waf.tp6.b1> ping 10.6.3.11

10.6.3.11 icmp_seq=1 timeout
84 bytes from 10.6.3.11 icmp_seq=2 ttl=62 time=40.823 ms
84 bytes from 10.6.3.11 icmp_seq=3 ttl=62 time=37.708 ms
84 bytes from 10.6.3.11 icmp_seq=4 ttl=62 time=32.570 ms
^C
waf.tp6.b1> ping 10.6.41.1

84 bytes from 10.6.41.1 icmp_seq=1 ttl=254 time=29.899 ms
84 bytes from 10.6.41.1 icmp_seq=2 ttl=254 time=24.158 ms
^C
waf.tp6.b1> ping 10.6.21.2

84 bytes from 10.6.21.2 icmp_seq=1 ttl=253 time=51.048 ms
84 bytes from 10.6.21.2 icmp_seq=2 ttl=253 time=51.698 ms
^C

meo.tp6.b1> ping 10.6.1.253

10.6.1.253 icmp_seq=1 timeout
10.6.1.253 icmp_seq=2 timeout
10.6.1.253 icmp_seq=3 timeout
10.6.1.253 icmp_seq=4 timeout

R5#ping 10.6.2.11

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.6.2.11, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 28/55/76 ms

R4#ping 10.6.21.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.6.21.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 8/16/24 ms

[manon@dhcp network-scripts]$ ping 10.6.21.1
PING 10.6.21.1 (10.6.21.1) 56(84) bytes of data.
64 bytes from 10.6.21.1: icmp_seq=1 ttl=254 time=25.6 ms
64 bytes from 10.6.21.1: icmp_seq=2 ttl=254 time=23.2 ms

waf.tp6.b1> ping google.com
google.com resolved to 142.250.179.78

R2#ping 1.1.1.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 64/119/208 ms
```

🦈 **`tp6_ospf.pcapng`**

- capturez des BPDUs là où vous voulez
- interprétez les BPDUs

> *Un BPDU c'est juste le nom qu'on donne à une trame OSPF échangée entre deux routeurs.*

[capture OSPF](./capture/tp6_ospf.pcapng)

## III. DHCP relay

➜ **Un problème très récurrent pour pas dire omniprésent avec DHCP c'est que ça marche que dans un LAN.**

Si t'as un serveur DHCP, et plein de réseaux comme c'est le cas ici, c'est le bordel :

- un DHCP Request, qui part en broadcast ne passe pas un routeur
- en effet, pour changer de réseau, il faut construire des paquets IP
- hors quand tu fais ton DHCP Request c'est ça que tu cherches : avoir une IP
- dans notre topo actuelle, impossible que John contacte le serveur DHCP

➜ **DHCP Relay !**

- on va demander à un routeur, s'il reçoit des trames DHCP de les faire passer vers notre serveur DHCP
- si le serveur DHCP le supporte, il répondra donc au routeur, qui fera passer au client

🌞 **Configurer un serveur DHCP** sur `dhcp.tp6.b1`

- même setup que d'habitude [(c'est ce lien que tu cherches ?)](https://www.server-world.info/en/note?os=Rocky_Linux_8&p=dhcp&f=1)
- votre serveur DHCP donne des IPs dans les réseaux
  - `10.6.1.0/24`
    - de `10.6.1.100` à `10.6.1.200`
    - informe les clients de l'adresse de la passerelle de ce réseau
    - informe les clients de l'adresse d'un serveur DNS : `1.1.1.1`
  - `10.6.3.0/24`
    - de `10.6.3.100` à `10.6.3.200`
    - informe les clients de l'adresse de la passerelle de ce réseau
    - informe les clients de l'adresse d'un serveur DNS : `1.1.1.1`
- pour le compte-rendu ça me suffit :
  - `sudo cat /etc/dhcp/dhcpd.conf`
  - `systemctl status dhcpd`

```bash
[manon@dhcp ~]$ sudo cat /etc/dhcp/dhcpd.conf
#
# DHCP Server Configuration file.
#   see /usr/share/doc/dhcp-server/dhcpd.conf.example
#   see dhcpd.conf(5) man page
# create new
# specify domain name
option domain-name     "srv.world";
# specify DNS server's hostname or IP address
option domain-name-servers     1.1.1.1;
# default lease time
default-lease-time 600;
# max lease time
max-lease-time 7200;
# this DHCP server to be declared valid
authoritative;
# specify network address and subnetmask
subnet 10.6.1.0 netmask 255.255.255.0 {
    # specify the range of lease IP address
    range dynamic-bootp 10.6.1.100 10.6.1.200;
    # specify broadcast address
    option broadcast-address 10.6.1.255;
    # specify gateway
    option routers 10.6.52.1;
}
subnet 10.6.3.0 netmask 255.255.255.0 {
    range dynamic-bootp 10.6.3.100 10.6.3.200;
    option broadcast-address 10.6.3.255;
    option routers 10.6.52.1;
}
```
```bash
[manon@dhcp ~]$ systemctl status dhcpd
● dhcpd.service - DHCPv4 Server Daemon
     Loaded: loaded (/usr/lib/systemd/system/dhcpd.service; enabled; preset: disabled)
     Active: active (running) since Wed 2023-12-06 18:47:54 CET; 48s ago
       Docs: man:dhcpd(8)
             man:dhcpd.conf(5)
   Main PID: 11510 (dhcpd)
     Status: "Dispatching packets..."
      Tasks: 1 (limit: 11039)
     Memory: 4.6M
        CPU: 7ms
     CGroup: /system.slice/dhcpd.service
             └─11510 /usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd.conf -user dhcpd -group dhcpd --no-pid

Dec 06 18:47:54 dhcp.tp6.b1 dhcpd[11510]:
Dec 06 18:47:54 dhcp.tp6.b1 dhcpd[11510]: No subnet declaration for enp0s3 (no IPv4 addresses).
Dec 06 18:47:54 dhcp.tp6.b1 dhcpd[11510]: ** Ignoring requests on enp0s3.  If this is not what
Dec 06 18:47:54 dhcp.tp6.b1 dhcpd[11510]:    you want, please write a subnet declaration
Dec 06 18:47:54 dhcp.tp6.b1 dhcpd[11510]:    in your dhcpd.conf file for the network segment
Dec 06 18:47:54 dhcp.tp6.b1 dhcpd[11510]:    to which interface enp0s3 is attached. **
Dec 06 18:47:54 dhcp.tp6.b1 dhcpd[11510]:
Dec 06 18:47:54 dhcp.tp6.b1 dhcpd[11510]: Sending on   Socket/fallback/fallback-net
Dec 06 18:47:54 dhcp.tp6.b1 dhcpd[11510]: Server starting service.
Dec 06 18:47:54 dhcp.tp6.b1 systemd[1]: Started DHCPv4 Server Daemon.
```

🌞 **Configurer un DHCP relay sur la passerelle de John**

- vérifier que Waf et John peuvent récupérer une IP en DHCP
- check les logs du serveur DHCP pour voir les DORA
  - je veux ces 4 lignes de logs dans le compte-rendu
  - pour John et pour Waf
- la conf sur le routeur qui est la passerelle de John c'est :

```cisco
R1#conf t
R1(config)#interface fastEthernet 2/0 # interface qui va recevoir des requêtes DHCP
R1(config-if)#ip helper-address <DHCP_SERVER_IP_ADDRESS>
```

```bash
[manon@dhcp ~]$ sudo cat /var/log/messages | grep john
Dec  6 22:08:26 dhcp dhcpd[808]: DHCPDISCOVER from 00:50:79:66:68:02 (john.tp6.b1) via 10.6.3.254
Dec  6 22:08:26 dhcp dhcpd[808]: DHCPOFFER on 10.6.3.100 to 00:50:79:66:68:02 (john.tp6.b1) via 10.6.3.254
Dec  6 22:08:29 dhcp dhcpd[808]: DHCPREQUEST for 10.6.3.100 (10.6.1.253) from 00:50:79:66:68:02 (john.tp6.b1) via 10.6.3.254
Dec  6 22:08:29 dhcp dhcpd[808]: DHCPACK on 10.6.3.100 to 00:50:79:66:68:02 (john.tp6.b1) via 10.6.3.254

[manon@dhcp ~]$ sudo cat /var/log/messages | grep waf
Dec  6 22:16:03 dhcp dhcpd[808]: DHCPDISCOVER from 00:50:79:66:68:00 (waf.tp6.b1) via enp0s8
Dec  6 22:16:03 dhcp dhcpd[808]: DHCPOFFER on 10.6.1.100 to 00:50:79:66:68:00 (waf.tp6.b1) via enp0s8
Dec  6 22:16:04 dhcp dhcpd[808]: DHCPREQUEST for 10.6.1.100 (10.6.1.253) from 00:50:79:66:68:00 (waf.tp6.b1) via enp0s8
Dec  6 22:16:04 dhcp dhcpd[808]: DHCPACK on 10.6.1.100 to 00:50:79:66:68:00 (waf.tp6.b1) via enp0s8
```

## IV. Bonus

### 1. ACL

C'est un peu moche que les clients puissent `ping` les IPs des routeurs de l'autre côté de l'infra.

Normalement, il peut joindre sa passerelle, internet, épuicétou.

🌞 **Configurer une access-list**

- ça se fait sur les routeurs
- le but :
  - les clients peuvent ping leur passerelle
  - et internet
  - épuicétou

```bash
R1(config)#access-list 1 permit 10.6.52.1 0.0.0.3
R1(config)#access-list 1 deny any

R1(config)#interface fastEthernet0/0
R1(config-if)#ip access-group 1 out
R1(config-if)#exit

R1(config)#interface fastEthernet0/1
R1(config-if)#ip access-group 1 out
R1(config-if)#exit
```
```bash
R2(config)#access-list 1 permit 10.6.52.1 0.0.0.3
R2(config)#access-list 1 deny any

R2(config)#interface fastEthernet1/0
R2(config-if)#ip access-group 1 out
R2(config-if)#exit
```
```bash
R3(config)#access-list 1 permit 10.6.52.1 0.0.0.3
R3(config)#access-list 1 deny any

R3(config)#interface fastEthernet0/1
R3(config-if)#ip access-group 1 out
R3(config-if)#exit
```
```bash
R4(config)#access-list 1 permit 10.6.52.1 0.0.0.3
R4(config)#access-list 1 deny any

R4(config)#interface fastEthernet0/0
R4(config-if)#ip access-group 1 out
```