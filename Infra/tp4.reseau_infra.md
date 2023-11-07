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

# 0. Prérequis

➜ Les clients seront soit :

- VMs Rocky Linux
- VPCS
  - c'est un truc de GNS pour simuler un client du réseau
  - quand on veut juste un truc capable de faire des pings et rien de plus, c'est parfait
  - ça consomme R en ressources

> Faites bien attention aux logos des machines sur les schémas, et vous verrez clairement quand il faut un VPCS ou une VM.

➜ Les switches Cisco des vIOL2 (IOU)

➜ Les routeurs Cisco des c3640

➜ **Vous ne créerez aucune machine virtuelle au début. Vous les créerez au fur et à mesure que le TP vous le demande.** A chaque fois qu'une nouvelle machine devra être créée, vous trouverez l'emoji 🖥️ avec son nom.

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
R1(config)#interface FastEthernet0/0
R1(config-if)#ip nat inside
R1(config-if)#exit
```
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
  10.1.20.0/24 via 10.1.30.254 dev enp0s8
  10.1.10.0/24 via 10.1.30.254 dev enp0s8
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
  
# [II. NAT](./2_nat/README.md)

# [III. Add a building](./3_second_building/README.md)