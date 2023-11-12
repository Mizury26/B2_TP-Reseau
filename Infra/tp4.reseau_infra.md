# TP4 : Router-on-a-stick

On va utiliser GNS3 dans ce TP pour se rapprocher d'un cas réel. La topologie qu vous mettez en place en fin de TP c'est tellement du classique qu'on lui a donné un nom : Router on a Stick.

Un seul routeur, plein de switches, plein de clients, plein de VLANs. Router on a Stick.

On va donc focus sur l'aspect routing/switching, avec du matériel Cisco. On va aussi mettre en place des VLANs, et du routage.

# Sommaire

- [TP4 : Router-on-a-stick](#tp4--router-on-a-stick)
- [Sommaire](#sommaire)
- [0. Prérequis](#0-prérequis)
  - [Checklist VM Linux](#checklist-vm-linux)
- [I. VLAN et Routing](#i-vlan-et-routing)
- [II. NAT](#ii-nat)
- [III. Add a building](#iii-add-a-building)


## Checklist VM Linux

A chaque machine déployée, vous **DEVREZ** vérifier la 📝**checklist**📝 :

- [x] IP locale, statique ou dynamique
- [x] hostname défini
- [x] firewall actif, qui ne laisse passer que le strict nécessaire
- [x] on force une host-only, juste pour pouvoir SSH
- [x] SSH fonctionnel
- [x] résolution de nom
  - vers internet, quand vous aurez le routeur en place

**Les éléments de la 📝checklist📝 sont STRICTEMENT OBLIGATOIRES à réaliser mais ne doivent PAS figurer dans le rendu.**

## 1. Topologie 1

## 2. Adressage topologie 1

Les réseaux et leurs VLANs associés :

| Réseau    | Adresse        | VLAN associé |
| --------- | -------------- | ------------ |
| `clients` | `10.1.10.0/24` | 10           |
| `admins`  | `10.1.20.0/24` | 20           |
| `servers` | `10.1.30.0/24` | 30           |

L'adresse des machines au sein de ces réseaux :

| Node               | `clients`       | `admins`         | `servers`        |
| ------------------ | --------------- | ---------------- | ---------------- |
| `pc1.clients.tp4`  | `10.1.10.1/24`   | x                | x                |
| `pc2.clients.tp4`  | `10.1.10.2/24`   | x                | x                |
| `adm1.admins.tp4`  | x               | `10.1.20.1/24`   | x                |
| `web1.servers.tp4` | x               | x                | `10.1.30.1/24`   |
| `r1`               | `10.1.10.254/24` | `10.1.20.254/24` | `10.1.30.254/24` |

## 3. Setup topologie 1

🖥️ VM `web1.servers.tp4`, déroulez la [Checklist VM Linux](#checklist-vm-linux) dessus

🌞 **Adressage**

- définissez les IPs statiques sur toutes les machines **sauf le *routeur***

```bash
PC1> show ip

NAME        : PC1[1]
IP/MASK     : 10.1.10.1/24
```
```bash
PC2> show ip

NAME        : PC2[1]
IP/MASK     : 10.1.10.2/24
```
```bash
PC3> show ip

NAME        : PC3[1]
IP/MASK     : 10.1.20.1/24
```
```bash
[manon@web1 ~]$ ip a
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:dc:6a:b6 brd ff:ff:ff:ff:ff:ff
    inet 10.1.30.1/24 brd 10.1.30.255 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fedc:6ab6/64 scope link
       valid_lft forever preferred_lft forever
```

🌞 **Configuration des VLANs**

- déclaration des VLANs sur le switch `sw1`
- ajout des ports du switches dans le bon VLAN (voir [le tableau d'adressage de la topo 2 juste au dessus](#2-adressage-topologie-2))

```bash

IOU1#show vlan br

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et0/0, Et1/1, Et1/2, Et1/3
                                                Et2/0, Et2/1, Et2/2, Et2/3
                                                Et3/0, Et3/1, Et3/2, Et3/3
10   clients                          active    Et0/1, Et0/2
20   admins                           active    Et0/3
30   servers                          active    Et1/0
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup
```

- il faudra ajouter le port qui pointe vers le *routeur* comme un *trunk* : c'est un port entre deux équipements réseau (un *switch* et un *routeur*)
```bash
IOU1(config)#interface Ethernet0/0
IOU1(config-if)#switchport trunk encapsulation dot1q
IOU1(config-if)#switchport mode trunk
IOU1(config-if)#switchport trunk allow vlan add 10,20,30
IOU1(config-if)#exit
IOU1(config)#exit

IOU1#show interface trunk

Port        Mode             Encapsulation  Status        Native vlan
Et0/0       on               802.1q         trunking      1

Port        Vlans allowed on trunk
Et0/0       1-4094

Port        Vlans allowed and active in management domain
Et0/0       1,10,20,30

Port        Vlans in spanning tree forwarding state and not pruned
Et0/0       1,10,20,30
```
```bash
IOU1#copy running-config startup-config
Destination filename [startup-config]?
Building configuration...
Compressed configuration from 1687 bytes to 1000 bytes[OK]
```
---

➜ **Pour le *routeur***

- ici, on va avoir besoin d'un truc très courant pour un *routeur* : qu'il porte plusieurs IP sur une unique interface
  - avec Cisco, on crée des "sous-interfaces" sur une interface
  - et on attribue une IP à chacune de ces sous-interfaces
- en plus de ça, il faudra l'informer que, pour chaque interface, elle doit être dans un VLAN spécifique


🌞 **Config du *routeur***

- attribuez ses IPs au *routeur*
  - 3 sous-interfaces, chacune avec son IP et un VLAN associé
```bash
R1(config)#interface FastEthernet0/0.10
R1(config-subif)#encapsulation dot1Q 10
R1(config-subif)#ip addr 10.1.10.254 255.255.255.0
R1(config-subif)#exit
```
```bash
R1(config)#interface FastEthernet0/0.20
R1(config-subif)#encapsulation dot1Q 20
R1(config-subif)#ip addr 10.1.20.254 255.255.255.0
R1(config-subif)#exit
```
```bash
R1(config)#interface FastEthernet0/0.30
R1(config-subif)#encapsulation dot1Q 30
R1(config-subif)#ip addr 10.1.30.254 255.255.255.0
R1(config-subif)#exit
```
```bash
R1#show ip int br
Any interface listed with OK? value "NO" does not have a valid configuration

Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            unassigned      YES unset  administratively down down
FastEthernet0/0.10         10.1.10.254     YES manual administratively down down
FastEthernet0/0.20         10.1.20.254     YES manual administratively down down
FastEthernet0/0.30         10.1.30.254     YES manual administratively down down
FastEthernet0/1            unassigned      YES unset  administratively down down
FastEthernet1/0            unassigned      YES unset  administratively down down
FastEthernet2/0            unassigned      YES unset  administratively down down
NVI0                       unassigned      NO  unset  up                    up
```
```bash
R1(config)#interface FastEthernet0/0
R1(config-if)#no shut
R1(config-subif)#exit
R1#show ip int br
Any interface listed with OK? value "NO" does not have a valid configuration

Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            unassigned      YES unset  up                    up
FastEthernet0/0.10         10.1.10.254     YES manual up                    up
FastEthernet0/0.20         10.1.20.254     YES manual up                    up
FastEthernet0/0.30         10.1.30.254     YES manual up                    up
FastEthernet0/1            unassigned      YES unset  administratively down down
FastEthernet1/0            unassigned      YES unset  administratively down down
FastEthernet2/0            unassigned      YES unset  administratively down down
NVI0                       unassigned      NO  unset  up                    up
```

🌞 **Vérif**

- tout le monde doit pouvoir ping le routeur sur l'IP qui est dans son réseau
```bash
PC1> ping 10.1.10.254

84 bytes from 10.1.10.254 icmp_seq=1 ttl=255 time=9.836 ms
84 bytes from 10.1.10.254 icmp_seq=2 ttl=255 time=3.090 ms

```
```bash

PC2> ping 10.1.10.254

84 bytes from 10.1.10.254 icmp_seq=1 ttl=255 time=9.145 ms
84 bytes from 10.1.10.254 icmp_seq=2 ttl=255 time=6.812 ms

```
```bash
adm1> ping 10.1.20.254

84 bytes from 10.1.20.254 icmp_seq=1 ttl=255 time=8.994 ms
84 bytes from 10.1.20.254 icmp_seq=2 ttl=255 time=10.674 ms

```
```bash
[manon@web1 ~]$ ping 10.1.30.254
PING 10.1.30.254 (10.1.30.254) 56(84) bytes of data.
64 bytes from 10.1.30.254: icmp_seq=1 ttl=255 time=16.9 ms
64 bytes from 10.1.30.254: icmp_seq=2 ttl=255 time=7.04 ms
^C
--- 10.1.30.254 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 7.043/11.960/16.878/4.917 ms
```
- en ajoutant une route vers les réseaux, ils peuvent se ping entre eux
  - ajoutez une route par défaut sur les VPCS
  ```bash
  PC1> ip 10.1.10.1 255.255.255.0 10.1.10.254
  Checking for duplicate address...
  PC1 : 10.1.10.1 255.255.255.0 gateway 10.1.10.254

  PC1> sh ip

  NAME        : PC1[1]
  IP/MASK     : 10.1.10.1/24
  GATEWAY     : 10.1.10.254
  DNS         :
  MAC         : 00:50:79:66:68:00
  LPORT       : 20015
  RHOST:PORT  : 127.0.0.1:20016
  MTU         : 1500
  
  PC1> save
  Saving startup configuration to startup.vpc
  .  done
  ```
  ```bash

  PC2> ip 10.1.10.2 255.255.255.0 10.1.10.254
  Checking for duplicate address...
  PC2 : 10.1.10.2 255.255.255.0 gateway 10.1.10.254
  
  PC2> sh ip

  NAME        : PC2[1]
  IP/MASK     : 10.1.10.2/24
  GATEWAY     : 10.1.10.254
  DNS         :
  MAC         : 00:50:79:66:68:01
  LPORT       : 20013
  RHOST:PORT  : 127.0.0.1:20014
  MTU         : 1500

  PC2> save
  Saving startup configuration to startup.vpc
  .  done
  ```
  ```bash
  adm1> ip 10.1.20.1 255.255.255.0 10.1.20.254
  Checking for duplicate address...
  adm1 : 10.1.20.1 255.255.255.0 gateway 10.1.20.254

  adm1> sh ip

  NAME        : adm1[1]
  IP/MASK     : 10.1.20.1/24
  GATEWAY     : 10.1.20.254
  DNS         :
  MAC         : 00:50:79:66:68:02
  LPORT       : 20011
  RHOST:PORT  : 127.0.0.1:20012
  MTU         : 1500

  adm1> save
  Saving startup configuration to startup.vpc
  .  done
  ```
  - ajoutez une route par défaut sur la machine virtuelle
  ```bash
  [manon@web1 ~]$ cat /etc/sysconfig/network-scripts/route-enp0s8
  default via 10.1.30.254 dev enp0s8
  ```
  - testez des `ping` entre les réseaux
  ```bash
  PC1> ping 10.1.20.1

  84 bytes from 10.1.20.1 icmp_seq=1 ttl=63 time=28.114 ms
  84 bytes from 10.1.20.1 icmp_seq=2 ttl=63 time=19.389 ms
  
  PC1> ping 10.1.30.1

  84 bytes from 10.1.30.1 icmp_seq=1 ttl=63 time=19.241 ms
  84 bytes from 10.1.30.1 icmp_seq=2 ttl=63 time=19.910 ms
  ```
  ```bash
  PC2> ping 10.1.20.1

  84 bytes from 10.1.20.1 icmp_seq=1 ttl=63 time=31.544 ms
  84 bytes from 10.1.20.1 icmp_seq=2 ttl=63 time=18.777 ms

  PC2> ping 10.1.30.1

  84 bytes from 10.1.30.1 icmp_seq=1 ttl=63 time=20.682 ms
  84 bytes from 10.1.30.1 icmp_seq=2 ttl=63 time=16.870 ms 
  ```
  ```bash
  adm1> ping 10.1.10.1

  84 bytes from 10.1.10.1 icmp_seq=1 ttl=63 time=21.173 ms
  84 bytes from 10.1.10.1 icmp_seq=2 ttl=63 time=19.707 ms

  adm1> ping 10.1.10.2

  84 bytes from 10.1.10.2 icmp_seq=1 ttl=63 time=15.328 ms
  84 bytes from 10.1.10.2 icmp_seq=2 ttl=63 time=11.172 ms

  adm1> ping 10.1.30.1

  84 bytes from 10.1.30.1 icmp_seq=1 ttl=63 time=17.646 ms
  84 bytes from 10.1.30.1 icmp_seq=2 ttl=63 time=17.475 ms
  ```
  ```bash
  [manon@web1 ~]$ ping 10.1.10.1
  PING 10.1.10.1 (10.1.10.1) 56(84) bytes of data.
  64 bytes from 10.1.10.1: icmp_seq=1 ttl=63 time=23.2 ms
  64 bytes from 10.1.10.1: icmp_seq=2 ttl=63 time=15.5 ms
  ^C
  --- 10.1.10.1 ping statistics ---
  2 packets transmitted, 2 received, 0% packet loss, time 1003ms
  rtt min/avg/max/mdev = 15.517/19.345/23.173/3.828 ms

  [manon@web1 ~]$ ping 10.1.10.2
  PING 10.1.10.2 (10.1.10.2) 56(84) bytes of data.
  64 bytes from 10.1.10.2: icmp_seq=1 ttl=63 time=30.7 ms
  64 bytes from 10.1.10.2: icmp_seq=2 ttl=63 time=17.9 ms
  ^C
  --- 10.1.10.2 ping statistics ---
  2 packets transmitted, 2 received, 0% packet loss, time 1003ms
  rtt min/avg/max/mdev = 17.916/24.310/30.705/6.394 ms

  [manon@web1 ~]$ ping 10.1.20.1
  PING 10.1.20.1 (10.1.20.1) 56(84) bytes of data.
  64 bytes from 10.1.20.1: icmp_seq=1 ttl=63 time=27.4 ms
  64 bytes from 10.1.20.1: icmp_seq=2 ttl=63 time=17.7 ms
  ^C
  --- 10.1.20.1 ping statistics ---
  2 packets transmitted, 2 received, 0% packet loss, time 1002ms
  rtt min/avg/max/mdev = 17.746/22.562/27.379/4.816 ms
  ```
# II. NAT

On va ajouter une fonctionnalité au routeur : le NAT.

On va le connecter à internet (simulation du fait d'avoir une IP publique) et il va faire du NAT pour permettre à toutes les machines du réseau d'avoir un accès internet.

## 1. Topologie 2

## 2. Setup topologie 2

🌞 **Ajoutez le noeud Cloud à la topo**

- branchez à `eth1` côté Cloud
- côté routeur, il faudra récupérer une IP en DHCP (voir [le mémo Cisco](../../../../cours/memo/memo_cisco.md))
```bash
R1(config)#interface FastEthernet1/0
R1(config-if)#ip address dhcp
R1(config-if)#no shut


R1#show ip int br
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            unassigned      YES unset  up                    up
FastEthernet0/0.10         10.1.10.254     YES manual up                    up
FastEthernet0/0.20         10.1.20.254     YES manual up                    up
FastEthernet0/0.30         10.1.30.254     YES manual up                    up
FastEthernet0/0.120        unassigned      YES unset  up                    up
FastEthernet0/1            unassigned      YES unset  administratively down down
FastEthernet1/0            192.168.122.89  YES DHCP   up                    up
FastEthernet2/0            unassigned      YES unset  administratively down down
NVI0                       10.1.10.254     YES unset  up                    up

R1#copy running-config startup-config
Destination filename [startup-config]?
Building configuration...
[OK]
```
- vous devriez pouvoir `ping 1.1.1.1`
```bash
R1#ping 1.1.1.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 36/48/72 ms
```

🌞 **Configurez le NAT**

```bash
R1(config)#interface FastEthernet0/0
R1(config-if)#ip nat inside
R1(config-if)#exit

R1(config)#interface FastEthernet0/0.10
R1(config-if)#ip nat inside
R1(config-if)#exit

R1(config)#interface FastEthernet0/0.20
R1(config-if)#ip nat inside
R1(config-if)#exit

R1(config)#interface FastEthernet0/0.30
R1(config-if)#ip nat inside
R1(config-if)#exit

R1(config)#interface FastEthernet1/0
R1(config-if)#ip nat outside
R1(config-if)#exit

R1(config)#access-list 1 permit any

R1(config)#ip nat inside source list 1 interface FastEthernet0/1 overload

R1#copy running-config startup-config
Destination filename [startup-config]?
Building configuration...
[OK]
```

🌞 **Test**

- configurez l'utilisation d'un DNS
  - sur les VPCS
  ```bash
  PC1> ip dns 1.1.1.1
  PC2> ip dns 1.1.1.1
  adm1> ip dns 1.1.1.1
  ```

  - sur la machine Linux
  ```bash
  [manon@web1 ~]$ cat /etc/resolv.conf
  # Generated by NetworkManager
  search servers.tp4
  nameserver 8.8.8.8
  nameserver 1.1.1.1
  ```
- vérifiez un `ping` vers un nom de domaine

```bash
PC1> ping ynov.com
ynov.com resolved to 104.26.10.233

PC2> ping ynov.com
ynov.com resolved to 104.26.10.233

PC3> ping ynov.com
ynov.com resolved to 104.26.10.233

[manon@web1 ~]$ ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=2 ttl=54 time=37.1 ms
64 bytes from 1.1.1.1: icmp_seq=3 ttl=54 time=38.2 ms
```

# III. Add a building

On a acheté un nouveau bâtiment, faut tirer et configurer un nouveau switch jusque là-bas.

On va en profiter pour setup un serveur DHCP pour les clients qui s'y trouvent.

## 1. Topologie 3

## 2. Adressage topologie 3

Les réseaux et leurs VLANs associés :

| Réseau    | Adresse        | VLAN associé |
| --------- | -------------- | ------------ |
| `clients` | `10.1.10.0/24`  | 10           |
| `admins`  | `10.1.20.0/24` | 20           |
| `servers` | `10.1.30.0/24` | 30           |

L'adresse des machines au sein de ces réseaux :

| Node                | `clients`       | `admins`         | `servers`        |
| ------------------- | --------------- | ---------------- | ---------------- |
| `pc1.clients.tp4`   | `10.1.10.1/24`   | x                | x                |
| `pc2.clients.tp4`   | `10.1.10.2/24`   | x                | x                |
| `pc3.clients.tp4`   | DHCP            | x                | x                |
| `pc4.clients.tp4`   | DHCP            | x                | x                |
| `pc5.clients.tp4`   | DHCP            | x                | x                |
| `dhcp1.clients.tp4` | `10.1.10.253/24` | x                | x                |
| `adm1.admins.tp4`   | x               | `10.1.20.1/24`   | x                |
| `web1.servers.tp4`  | x               | x                | `10.1.30.1/24`   |
| `r1`                | `10.1.10.254/24` | `10.1.20.254/24` | `10.1.30.254/24` |

## 3. Setup topologie 3

Vous pouvez partir de la topologie 2.

🌞  **Vous devez me rendre le `show running-config` de tous les équipements**

- de tous les équipements réseau
  - le routeur
```bash
R1#show run
Building configuration...

Current configuration : 1630 bytes
!
version 12.4
service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption
!
hostname R1
!
boot-start-marker
boot-end-marker
!
!
no aaa new-model
memory-size iomem 5
no ip icmp rate-limit unreachable
ip cef
!
!
!
!
no ip domain lookup
!
multilink bundle-name authenticated
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!
!
archive
 log config
  hidekeys
!
!
!
!
ip tcp synwait-time 5
!
!
!
!
interface FastEthernet0/0
 no ip address
 ip nat inside
 ip virtual-reassembly
 speed auto
 full-duplex
!
interface FastEthernet0/0.10
 encapsulation dot1Q 10
 ip address 10.1.10.254 255.255.255.0
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet0/0.20
 encapsulation dot1Q 20
 ip address 10.1.20.254 255.255.255.0
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet0/0.30
 encapsulation dot1Q 30
 ip address 10.1.30.254 255.255.255.0
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet0/1
 no ip address
 shutdown
 duplex auto
 speed auto
!
interface FastEthernet1/0
 ip address dhcp
 ip nat outside
 ip virtual-reassembly
 duplex auto
 speed auto
!
interface FastEthernet2/0
 no ip address
 shutdown
 duplex auto
 speed auto
!
ip forward-protocol nd
!
!
no ip http server
no ip http secure-server
ip nat inside source list 1 interface FastEthernet1/0 overload
!
access-list 1 permit any
no cdp log mismatch duplex
!
!
!
!
!
!
control-plane
!
!
!
!
!
!
!
!
!
!
line con 0
 exec-timeout 0 0
 privilege level 15
 logging synchronous
line aux 0
 exec-timeout 0 0
 privilege level 15
 logging synchronous
line vty 0 4
 login
!
!
end
```

  - les 3 switches

  **Switch 1 :**
  ```bash
  IOU3#show run
  Building configuration...

  Current configuration : 1484 bytes
  !
  ! Last configuration change at 13:28:44 UTC Sun Nov 12 2023
  !
  version 15.2
  service timestamps debug datetime msec
  service timestamps log datetime msec
  no service password-encryption
  service compress-config
  !
  hostname IOU3
  !
  boot-start-marker
  boot-end-marker
  !
  !
  logging discriminator EXCESS severity drops 6 msg-body drops  EXCESSCOLL
  logging buffered 50000
  logging console discriminator EXCESS
  !
  no aaa new-model
  !
  !
  !
  !
  !
  no ip icmp rate-limit unreachable
  !
  !
  !
  no ip domain-lookup
  ip cef
  no ipv6 cef
  !
  !
  !
  spanning-tree mode pvst
  spanning-tree extend system-id
  !
  !
  !
  !
  !
  !
  !
  !
  !
  !
  !
  !
  !
  !
  !
  interface Ethernet0/0
   switchport trunk encapsulation dot1q
   switchport mode trunk
  !
  interface Ethernet0/1
  !
  interface Ethernet0/2
  !
  interface Ethernet0/3
  !
  interface Ethernet1/0
  !
  interface Ethernet1/1
  !
  interface Ethernet1/2
  !
  interface Ethernet1/3
  !
  interface Ethernet2/0
  !
  interface Ethernet2/1
  !
  interface Ethernet2/2
  !
  interface Ethernet2/3
  !
  interface Ethernet3/0
  !
  interface Ethernet3/1
  !
  interface Ethernet3/2
  !
  interface Ethernet3/3
  !
  interface Vlan1
   no ip address
   shutdown
  !
  ip forward-protocol nd
  !
  ip tcp synwait-time 5
  ip http server
  !
  ip ssh server algorithm encryption aes128-ctr aes192-ctr  aes256-ctr
  ip ssh client algorithm encryption aes128-ctr aes192-ctr  aes256-ctr
  !
  !
  !
  !
  !
  control-plane
  !
  !
  line con 0
   exec-timeout 0 0
   privilege level 15
   logging synchronous
  line aux 0
   exec-timeout 0 0
   privilege level 15
   logging synchronous
  line vty 0 4
   login
  !
  !
  !
  end
  ```
  **Switch :**
  ```bash
  IOU1#show run
  Building configuration...

  Current configuration : 1739 bytes
  !
  ! Last configuration change at 12:59:25 UTC Sun Nov 12 2023
  !
  version 15.2
  service timestamps debug datetime msec
  service timestamps log datetime msec
  no service password-encryption
  service compress-config
  !
  hostname IOU1
  !
  boot-start-marker
  boot-end-marker
  !
  !
  logging discriminator EXCESS severity drops 6 msg-body drops  EXCESSCOLL
  logging buffered 50000
  logging console discriminator EXCESS
  !
  no aaa new-model
  !
  !
  !
  !
  !
  no ip icmp rate-limit unreachable
  !
  !
  !
  no ip domain-lookup
  ip cef
  no ipv6 cef
  !
  !
  !
  spanning-tree mode pvst
  spanning-tree extend system-id
  !
  !
  !
  !
  !
  !
  !
  !
  !
  !
  !
  !
  !
  !
  !
  interface Ethernet0/0
   switchport trunk encapsulation dot1q
   switchport mode trunk
  !
  interface Ethernet0/1
   switchport access vlan 10
   switchport mode access
  !
  interface Ethernet0/2
   switchport access vlan 10
   switchport mode access
  !
  interface Ethernet0/3
   switchport access vlan 20
   switchport mode access
  !
  interface Ethernet1/0
   switchport access vlan 30
   switchport mode access
  !
  interface Ethernet1/1
   switchport access vlan 30
   switchport mode access
  !
  interface Ethernet1/2
  !
  interface Ethernet1/3
  !
  interface Ethernet2/0
  !
  interface Ethernet2/1
  !
  interface Ethernet2/2
  !
  interface Ethernet2/3
  !
  interface Ethernet3/0
  !
  interface Ethernet3/1
  !
  interface Ethernet3/2
  !
  interface Ethernet3/3
  !
  interface Vlan1
   no ip address
   shutdown
  !
  ip forward-protocol nd
  !
  ip tcp synwait-time 5
  ip http server
  !
  ip ssh server algorithm encryption aes128-ctr aes192-ctr  aes256-ctr
  ip ssh client algorithm encryption aes128-ctr aes192-ctr  aes256-ctr
  !
  !
  !
  !
  !
  control-plane
  !
  !
  line con 0
   exec-timeout 0 0
   privilege level 15
   logging synchronous
  line aux 0
   exec-timeout 0 0
   privilege level 15
   logging synchronous
  line vty 0 4
   login
  !
  !
  !
  end
  ```
  **Switch 3 :**
  ```bash
  IOU2#show run
  Building configuration...

  Current configuration : 1637 bytes
  !
  ! Last configuration change at 13:45:13 UTC Sun Nov 12 2023
  !
  version 15.2
  service timestamps debug datetime msec
  service timestamps log datetime msec
  no service password-encryption
  service compress-config
  !
  hostname IOU2
  !
  boot-start-marker
  boot-end-marker
  !
  !
  logging discriminator EXCESS severity drops 6 msg-body drops  EXCESSCOLL
  logging buffered 50000
  logging console discriminator EXCESS
  !
  no aaa new-model
  !
  !
  !
  !
  !
  no ip icmp rate-limit unreachable
  !
  !
  !
  no ip domain-lookup
  ip cef
  no ipv6 cef
  !
  !
  !
  spanning-tree mode pvst
  spanning-tree extend system-id
  !
  !
  !
  !
  !
  !
  !
  !
  !
  !
  !
  !
  !
  !
  !
  interface Ethernet0/0
   switchport trunk encapsulation dot1q
   switchport mode trunk
  !
  interface Ethernet0/1
   switchport access vlan 10
   switchport mode access
  !
  interface Ethernet0/2
   switchport access vlan 10
   switchport mode access
  !
  interface Ethernet0/3
   switchport access vlan 10
   switchport mode access
  !
  interface Ethernet1/0
  switchport access vlan 10
  switchport mode access
  !
  interface Ethernet1/1
  !
  interface Ethernet1/2
  !
  interface Ethernet1/3
  !
  interface Ethernet2/0
  !
  interface Ethernet2/1
  !
  interface Ethernet2/2
  !
  interface Ethernet2/3
  !
  interface Ethernet3/0
  !
  interface Ethernet3/1
  !
  interface Ethernet3/2
  !
  interface Ethernet3/3
  !
  interface Vlan1
   no ip address
   shutdown
  !
  ip forward-protocol nd
  !
  ip tcp synwait-time 5
  ip http server
  !
  ip ssh server algorithm encryption aes128-ctr aes192-ctr  aes256-ctr
  ip ssh client algorithm encryption aes128-ctr aes192-ctr  aes256-ctr
  !
  !
  !
  !
  !
  control-plane
  !
  !
  line con 0
   exec-timeout 0 0
   privilege level 15
   logging synchronous
  line aux 0
   exec-timeout 0 0
   privilege level 15
   logging synchronous
  line vty 0 4
   login
  !
  !
  !
  end
  ```


🌞  **Mettre en place un serveur DHCP dans le nouveau bâtiment**

```bash
[manon@dhcp ~]$ sudo dnf install dhcp-server -y

[manon@dhcp ~]$ sudo cat /etc/dhcp/dhcpd.conf
# DHCP Server Configuration file.
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
subnet 10.1.10.0 netmask 255.255.255.0 {
    # specify the range of lease IP address
    range dynamic-bootp 10.1.10.3 10.1.10.252;
    # specify broadcast address
    option broadcast-address 10.1.10.255;
    # specify gateway
    option routers 10.1.10.254;
  }
```
```bash
[manon@dhcp ~]$ sudo systemctl enable --now dhcpd
[sudo] password for manon:
Created symlink /etc/systemd/system/multi-user.target.wants/dhcpd.service → /usr/lib/systemd/system/dhcpd.service.
[manon@dhcp ~]$ sudo firewall-cmd --add-service=dhcp
success
[manon@dhcp ~]$ sudo firewall-cmd --runtime-to-permanent
success
[manon@dhcp ~]$ systemctl status dhcpd
● dhcpd.service - DHCPv4 Server Daemon
     Loaded: loaded (/usr/lib/systemd/system/dhcpd.service; enabled; preset: disabled)
     Active: active (running) since Sun 2023-11-12 16:38:14 CET; 17s ago
       Docs: man:dhcpd(8)
             man:dhcpd.conf(5)
   Main PID: 1355 (dhcpd)
     Status: "Dispatching packets..."
      Tasks: 1 (limit: 11052)
     Memory: 7.1M
        CPU: 11ms
     CGroup: /system.slice/dhcpd.service
             └─1355 /usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd.conf -user dhcpd -group dhcpd --no-pid

Nov 12 16:38:14 dhcp.tp4.b2 dhcpd[1355]: ** Ignoring requests on enp0s8.  If this is not what
Nov 12 16:38:14 dhcp.tp4.b2 dhcpd[1355]:    you want, please write a subnet declaration
Nov 12 16:38:14 dhcp.tp4.b2 dhcpd[1355]:    in your dhcpd.conf file for the network segment
Nov 12 16:38:14 dhcp.tp4.b2 dhcpd[1355]:    to which interface enp0s8 is attached. **
Nov 12 16:38:14 dhcp.tp4.b2 dhcpd[1355]:
Nov 12 16:38:14 dhcp.tp4.b2 dhcpd[1355]: Listening on LPF/enp0s3/08:00:27:fc:5e:8a/10.1.10.0/24
Nov 12 16:38:14 dhcp.tp4.b2 dhcpd[1355]: Sending on   LPF/enp0s3/08:00:27:fc:5e:8a/10.1.10.0/24
Nov 12 16:38:14 dhcp.tp4.b2 dhcpd[1355]: Sending on   Socket/fallback/fallback-net
Nov 12 16:38:14 dhcp.tp4.b2 dhcpd[1355]: Server starting service.
Nov 12 16:38:14 dhcp.tp4.b2 systemd[1]: Started DHCPv4 Server Daemon.
```

- il doit distribuer des IPs aux clients dans le réseau `clients` qui sont branchés au même switch que lui
- sans aucune action manuelle, les clients doivent...
  - avoir une IP dans le réseau `clients`
  - avoir un accès au réseau `servers`
  - avoir un accès WAN
  - avoir de la résolution DNS


🌞  **Vérification**

- un client récupère une IP en DHCP
- il peut ping le serveur Web
- il peut ping `1.1.1.1`
- il peut ping `ynov.com`

> Faites ça sur n'importe quel VPCS que vous venez d'ajouter : `pc3` ou `pc4` ou `pc5`.

```bash
PC3> ip dhcp
DDORA IP 10.1.10.3/24 GW 10.1.10.254

PC3> sh ip

NAME        : PC3[1]
IP/MASK     : 10.1.10.3/24
GATEWAY     : 10.1.10.254
DNS         : 1.1.1.1
DHCP SERVER : 10.1.10.253
DHCP LEASE  : 594, 600/300/525
DOMAIN NAME : srv.world
MAC         : 00:50:79:66:68:0a
LPORT       : 20029
RHOST:PORT  : 127.0.0.1:20030
MTU         : 1500

PC3> ping 1.1.1.1

1.1.1.1 icmp_seq=1 timeout
^C
PC3> ping ynov.com
ynov.com resolved to 104.26.11.233

ynov.com icmp_seq=1 timeout
^C
PC3> ping 10.1.30.1

84 bytes from 10.1.30.1 icmp_seq=1 ttl=63 time=22.375 ms
84 bytes from 10.1.30.1 icmp_seq=2 ttl=63 time=18.063 ms
^C
```
