# TP2 : Environnement virtuel

Dans ce TP, on remanipule toujours les mêmes concepts qu'au TP1, mais en environnement virtuel avec une posture un peu plus orientée administrateur qu'au TP1.

- [TP2 : Environnement virtuel](#tp2--environnement-virtuel)
- [0. Prérequis](#0-prérequis)
- [I. Topologie réseau](#i-topologie-réseau)
  - [Topologie](#topologie)
  - [Tableau d'adressage](#tableau-dadressage)
  - [Hints](#hints)
  - [Marche à suivre recommandée](#marche-à-suivre-recommandée)
  - [Compte-rendu](#compte-rendu)
- [II. Interlude accès internet](#ii-interlude-accès-internet)
- [III. Services réseau](#iii-services-réseau)
  - [1. DHCP](#1-dhcp)
  - [2. Web web web](#2-web-web-web)

# 0. Prérequis

La même musique que l'an dernier :

- VirtualBox
- Rocky Linux
  - préparez une VM patron, prête à être clonée
  - système à jour (`dnf update`)
  - SELinux désactivé
  - préinstallez quelques paquets, je pense à notamment à :
    - `vim`
    - `bind-utils` pour la commande `dig`
    - `traceroute`
    - `tcpdump` pour faire des captures réseau

La ptite **checklist** que vous respecterez pour chaque VM :

- [ ] carte réseau host-only avec IP statique
- [ ] pas de carte NAT, sauf si demandée
- [ ] adresse IP statique sur la carte host-only
- [ ] connexion SSH fonctionnelle
- [ ] firewall actif
- [ ] SELinux désactivé
- [ ] hostname défini


# I. Topologie réseau

Vous allez dans cette première partie préparer toutes les VMs et vous assurez que leur connectivité réseau fonctionne bien.

On va donc parler essentiellement IP et routage ici.

## Topologie

## Tableau d'adressage

| Node             | LAN1 `10.1.1.0/24` | LAN2 `10.1.2.0/24` |
| ---------------- | ------------------ | ------------------ |
| `node1.lan1.tp1` | `10.1.1.11`        | x                  |
| `node2.lan1.tp1` | `10.1.1.12`        | x                  |
| `node1.lan2.tp1` | x                  | `10.1.2.11`        |
| `node2.lan2.tp1` | x                  | `10.1.2.12`        |
| `router.tp1`     | `10.1.1.254`       | `10.1.2.254`       |

## Hints

➜ **Sur le `router.tp1`**

Il sera nécessaire d'**activer le routage**. Par défaut Rocky n'agit pas comme un routeur. C'est à dire que par défaut il ignore les paquets qu'il reçoit s'il l'IP de destination n'est pas la sienne. Or, c'est précisément le job d'un routeur.

Pour activer le routage donc, sur une machine Rocky :

```bash
$ firewall-cmd --add-masquerade
$ firewall-cmd --add-masquerade --permanent
$ sysctl -w net.ipv4.ip_forward=1
```
```bash
[manon@router ~]$ sudo firewall-cmd --add-masquerade
[sudo] password for manon:
success
[manon@router ~]$ sudo firewall-cmd --add-masquerade --permanent
success
[manon@router ~]$ sudo sysctl -w net.ipv4.ip_forward=1
net.ipv4.ip_forward = 1
```
---

➜ **Aucune carte NAT**

## Marche à suivre recommandée

Dans l'ordre, je vous recommande de :

**1.** créer les VMs dans VirtualBox (clone du patron)  
**2.** attribuer des IPs statiques à toutes les VMs  
**3.** vous connecter en SSH à toutes les VMs  
**4.** activer le routage sur `router.tp1`  
**5.** vous assurer que les membres de chaque LAN se ping, c'est à dire :

- `node1.lan1.tp1`
  - doit pouvoir ping `node2.lan1.tp1`
  - doit aussi pouvoir ping `router.tp1` (il a deux IPs ce `router.tp1`, `node1.lan1.tp1` ne peut ping que celle qui est dans son réseau : `10.1.1.254`)
```bash
[manon@node1lan1 ~]$ ping 10.1.1.12
PING 10.1.1.12 (10.1.1.12) 56(84) bytes of data.
64 bytes from 10.1.1.12: icmp_seq=1 ttl=64 time=4.28 ms
64 bytes from 10.1.1.12: icmp_seq=2 ttl=64 time=3.43 ms
^C
--- 10.1.1.12 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 3.434/3.857/4.281/0.423 ms
[manon@node1lan1 ~]$ ping 10.1.1.254
PING 10.1.1.254 (10.1.1.254) 56(84) bytes of data.
64 bytes from 10.1.1.254: icmp_seq=1 ttl=64 time=6.41 ms
64 bytes from 10.1.1.254: icmp_seq=2 ttl=64 time=2.18 ms
^C
--- 10.1.1.254 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 2.176/4.294/6.412/2.118 ms
```
- `router.tp1` ping tout le monde
```bash
[manon@router ~]$ ping 10.1.1.1
PING 10.1.1.1 (10.1.1.1) 56(84) bytes of data.
64 bytes from 10.1.1.1: icmp_seq=1 ttl=128 time=1.76 ms
64 bytes from 10.1.1.1: icmp_seq=2 ttl=128 time=1.82 ms
^C
--- 10.1.1.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1003ms
rtt min/avg/max/mdev = 1.760/1.790/1.821/0.030 ms
[manon@router ~]$ ping 10.1.1.2
PING 10.1.1.2 (10.1.1.2) 56(84) bytes of data.
From 10.1.1.254 icmp_seq=1 Destination Host Unreachable
From 10.1.1.254 icmp_seq=2 Destination Host Unreachable
From 10.1.1.254 icmp_seq=3 Destination Host Unreachable
From 10.1.1.254 icmp_seq=4 Destination Host Unreachable
From 10.1.1.254 icmp_seq=5 Destination Host Unreachable
From 10.1.1.254 icmp_seq=6 Destination Host Unreachable
^C
--- 10.1.1.2 ping statistics ---
7 packets transmitted, 0 received, +6 errors, 100% packet loss, time 6141ms
pipe 4
[manon@router ~]$
[manon@router ~]$ ping 10.1.1.11
PING 10.1.1.11 (10.1.1.11) 56(84) bytes of data.
64 bytes from 10.1.1.11: icmp_seq=1 ttl=64 time=1.87 ms
64 bytes from 10.1.1.11: icmp_seq=2 ttl=64 time=2.24 ms
64 bytes from 10.1.1.11: icmp_seq=3 ttl=64 time=1.68 ms
64 bytes from 10.1.1.11: icmp_seq=4 ttl=64 time=2.42 ms
^C
--- 10.1.1.11 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3008ms
rtt min/avg/max/mdev = 1.682/2.052/2.420/0.291 ms
[manon@router ~]$
[manon@router ~]$ ping 10.1.1.11
PING 10.1.1.11 (10.1.1.11) 56(84) bytes of data.
64 bytes from 10.1.1.11: icmp_seq=1 ttl=64 time=2.56 ms
64 bytes from 10.1.1.11: icmp_seq=2 ttl=64 time=2.29 ms
^C
--- 10.1.1.11 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1003ms
rtt min/avg/max/mdev = 2.287/2.425/2.564/0.138 ms
[manon@router ~]$ ping 10.1.1.12
PING 10.1.1.12 (10.1.1.12) 56(84) bytes of data.
64 bytes from 10.1.1.12: icmp_seq=1 ttl=64 time=2.80 ms
64 bytes from 10.1.1.12: icmp_seq=2 ttl=64 time=1.78 ms
^C
--- 10.1.1.12 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1003ms
rtt min/avg/max/mdev = 1.778/2.289/2.800/0.511 ms
[manon@router ~]$ ping 10.1.2.11
PING 10.1.2.11 (10.1.2.11) 56(84) bytes of data.
64 bytes from 10.1.2.11: icmp_seq=1 ttl=64 time=5.02 ms
64 bytes from 10.1.2.11: icmp_seq=2 ttl=64 time=2.18 ms
^C
--- 10.1.2.11 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 2.179/3.601/5.024/1.422 ms
[manon@router ~]$ ping 10.1.2.12
PING 10.1.2.12 (10.1.2.12) 56(84) bytes of data.
64 bytes from 10.1.2.12: icmp_seq=1 ttl=64 time=3.27 ms
64 bytes from 10.1.2.12: icmp_seq=2 ttl=64 time=1.85 ms
^C
--- 10.1.2.12 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 1.850/2.562/3.274/0.712 ms
```
- les membres du LAN2 se ping aussi
```bash
[manon@node1lan2 ~]$ ping 10.1.2.11
PING 10.1.2.11 (10.1.2.11) 56(84) bytes of data.
64 bytes from 10.1.2.11: icmp_seq=1 ttl=64 time=0.081 ms
64 bytes from 10.1.2.11: icmp_seq=2 ttl=64 time=0.107 ms
^C
--- 10.1.2.11 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 0.081/0.094/0.107/0.013 ms
[manon@node1lan2 ~]$ ping 10.1.2.254
PING 10.1.2.254 (10.1.2.254) 56(84) bytes of data.
64 bytes from 10.1.2.254: icmp_seq=1 ttl=64 time=1.92 ms
64 bytes from 10.1.2.254: icmp_seq=2 ttl=64 time=1.98 ms
^C
--- 10.1.2.254 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 1.917/1.948/1.979/0.031 ms
```

**6.** ajouter les routes statiques

- sur les deux machines du LAN1, il faut ajouter une route vers le LAN2

```bash
[manon@node1lan1 ~]$ cat /etc/sysconfig/network-scripts/route-enp0s8
10.1.2.0/24 via 10.1.1.254 dev enp0s8
[manon@node1lan1 ~]$ ip route show
10.1.1.0/24 dev enp0s8 proto kernel scope link src 10.1.1.11 metric 100
10.1.2.0/24 via 10.1.1.254 dev enp0s8 proto static metric 100
[manon@node2lan1 ~]$ cat /etc/sysconfig/network-scripts/route-enp0s8
10.1.2.0/24 via 10.1.1.254 dev enp0s8
[manon@node2lan1 ~]$ ip route show
10.1.1.0/24 dev enp0s8 proto kernel scope link src 10.1.1.12 metric 100
10.1.2.0/24 via 10.1.1.254 dev enp0s8 proto static metric 100
```
- sur les deux machines du LAN2, il faut ajouter une route vers le LAN1
```bash
[manon@node1lan2 ~]$  cat /etc/sysconfig/network-scripts/route-enp0s8
10.1.1.0/24 via 10.1.2.254 dev enp0s8
[manon@node1lan2 ~]$ ip route show
10.1.1.0/24 via 10.1.2.254 dev enp0s8 proto static metric 100
10.1.2.0/24 dev enp0s8 proto kernel scope link src 10.1.2.11 metric 100
[manon@node2lan2 ~]$  cat /etc/sysconfig/network-scripts/route-enp0s8
10.1.1.0/24 via 10.1.2.254 dev enp0s8
[manon@node2lan2 ~]$ ip route show
10.1.1.0/24 via 10.1.2.254 dev enp0s8 proto static metric 100
10.1.2.0/24 dev enp0s8 proto kernel scope link src 10.1.2.12 metric 100
```

## Compte-rendu

☀️ Sur **`node1.lan1.tp1`**

- afficher ses cartes réseau
```bash
[manon@node1lan1 ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:4e:6b:1a brd ff:ff:ff:ff:ff:ff
    inet 10.1.1.11/24 brd 10.1.1.255 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe4e:6b1a/64 scope link
       valid_lft forever preferred_lft forever
```
- afficher sa table de routage
```bash
[manon@node1lan1 ~]$ netstat -r
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
default         _gateway        0.0.0.0         UG        0 0          0 enp0s3
10.0.2.0        0.0.0.0         255.255.255.0   U         0 0          0 enp0s3
10.1.1.0        0.0.0.0         255.255.255.0   U         0 0          0 enp0s8
10.1.2.0        10.1.1.254      255.255.255.0   UG        0 0          0 enp0s8
```
- prouvez qu'il peut joindre `node2.lan2.tp2`
```bash
[manon@node1lan1 ~]$ ping 10.1.2.12
PING 10.1.2.12 (10.1.2.12) 56(84) bytes of data.
64 bytes from 10.1.2.12: icmp_seq=1 ttl=63 time=2.65 ms
64 bytes from 10.1.2.12: icmp_seq=2 ttl=63 time=1.69 ms
^C
--- 10.1.2.12 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1003ms
rtt min/avg/max/mdev = 1.694/2.171/2.648/0.477 ms
```
- prouvez avec un `traceroute` que le paquet passe bien par `router.tp1`
```bash
[manon@node1lan1 ~]$ traceroute 10.1.2.12
traceroute to 10.1.2.12 (10.1.2.12), 30 hops max, 60 byte packets
 1  10.1.1.254 (10.1.1.254)  5.589 ms  5.378 ms  5.070 ms
 2  10.1.2.12 (10.1.2.12)  4.844 ms !X  4.417 ms !X  4.104 ms !X
 ```

# II. Interlude accès internet

**On va donner accès internet à tout le monde.** Le routeur aura un accès internet, et permettra à tout le monde d'y accéder : il sera la passerelle par défaut des membres du LAN1 et des membres du LAN2.

**Ajoutez une carte NAT au routeur pour qu'il ait un accès internet.**

☀️ **Sur `router.tp1`**

- prouvez que vous avez un accès internet (ping d'une IP publique)
```bash
[manon@router ~]$ ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=56 time=17.5 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=56 time=12.7 ms
^C
--- 1.1.1.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 12.675/15.066/17.458/2.391 ms
```
- prouvez que vous pouvez résoudre des noms publics (ping d'un nom de domaine public)
```bash
[manon@router ~]$ ping google.com
PING google.com (142.250.179.110) 56(84) bytes of data.
64 bytes from par21s20-in-f14.1e100.net (142.250.179.110): icmp_seq=1 ttl=115 time=24.2 ms
64 bytes from par21s20-in-f14.1e100.net (142.250.179.110): icmp_seq=2 ttl=115 time=33.1 ms
^C
--- google.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1003ms
rtt min/avg/max/mdev = 24.228/28.675/33.123/4.447 ms
```

☀️ **Accès internet LAN1 et LAN2**

- ajoutez une route par défaut sur les deux machines du LAN1
```bash
[manon@node1lan1 ~]$ cat /etc/sysconfig/network
GATEWAY=10.1.1.254
[manon@node2lan1 ~]$ cat /etc/sysconfig/network
GATEWAY=10.1.1.254
```
- ajoutez une route par défaut sur les deux machines du LAN2
```bash
[manon@node1lan2 ~]$ cat /etc/sysconfig/network
GATEWAY=10.1.2.254
[manon@node2lan2 ~]$ cat /etc/sysconfig/network
GATEWAY=10.1.2.254
```
- configurez l'adresse d'un serveur DNS que vos machines peuvent utiliser pour résoudre des noms
```bash
[manon@node2lan1 ~]$ cat vim /etc/resolv.conf
nameserver 8.8.8.8
nameserver 1.1.1.1
```
- dans le compte-rendu, mettez-moi que la conf des points précédents sur `node2.lan1.tp1`
- prouvez que `node2.lan1.tp1` a un accès internet :
  - il peut ping une IP publique
  ```bash
  [manon@node2lan1 ~]$ ping 1.1.1.1
  PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
  64 bytes from 1.1.1.1: icmp_seq=1 ttl=55 time=13.3 ms
  64 bytes from 1.1.1.1: icmp_seq=2 ttl=55 time=12.9 ms
  ^C
  --- 1.1.1.1 ping statistics ---
  2 packets transmitted, 2 received, 0% packet loss, time 1002ms
  rtt min/avg/max/mdev = 12.876/13.092/13.309/0.216 ms
  ```
  - il peut ping un nom de domaine public
  ```bash
  [manon@node2lan1 ~]$ ping ynov.com
  PING ynov.com (104.26.10.233) 56(84) bytes of data.
  64 bytes from 104.26.10.233 (104.26.10.233): icmp_seq=1 ttl=55  time=12.2 ms
  64 bytes from 104.26.10.233 (104.26.10.233): icmp_seq=2 ttl=55  time=13.1 ms
  ^C
  --- ynov.com ping statistics ---
  2 packets transmitted, 2 received, 0% packet loss, time 1002ms
  rtt min/avg/max/mdev = 12.220/12.684/13.149/0.464 ms
  ```

# III. Services réseau

**Adresses IP et routage OK, maintenant, il s'agirait d'en faire quelque chose nan ?**

Dans cette partie, on va **monter quelques services orientés réseau** au sein de la topologie, afin de la rendre un peu utile que diable. Des machines qui se `ping` c'est rigolo mais ça sert à rien, des machines qui font des trucs c'est mieux.

## 1. DHCP

![Dora](./img/dora.jpg)

Petite **install d'un serveur DHCP** dans cette partie. Par soucis d'économie de ressources, on recycle une des machines précédentes : `node2.lan1.tp1` devient `dhcp.lan1.tp1`.

**Pour rappel**, un serveur DHCP, on en trouve un dans la plupart des LANs auxquels vous vous êtes connectés. Si quand tu te connectes dans un réseau, tu n'es pas **obligé** de saisir une IP statique à la mano, et que t'as un accès internet wala, alors il y a **forcément** un serveur DHCP dans le réseau qui t'a proposé une IP disponible.

> Le serveur DHCP a aussi pour rôle de donner, en plus d'une IP disponible, deux informations primordiales pour l'accès internet : l'adresse IP de la passerelle du réseau, et l'adresse d'un serveur DNS joignable depuis ce réseau.

**Dans notre TP, son rôle sera de proposer une IP libre à toute machine qui le demande dans le LAN1.**

> Vous pouvez vous référer à [ce lien](https://www.server-world.info/en/note?os=Rocky_Linux_8&p=dhcp&f=1) ou n'importe quel autre truc sur internet (je sais c'est du Rocky 8, m'enfin, la conf de ce serveur DHCP ça bouge pas trop).

---

Pour ce qui est de la configuration du serveur DHCP, quelques précisions :

- vous ferez en sorte qu'il propose des adresses IPs entre `10.1.1.100` et `10.1.1.200`
- vous utiliserez aussi une option DHCP pour indiquer aux clients que la passerelle du réseau est `10.1.1.254` : le routeur
- vous utiliserez aussi une option DHCP pour indiquer aux clients qu'un serveur DNS joignable depuis le réseau c'est `1.1.1.1`

---

☀️ **Sur `dhcp.lan1.tp1`**

- n'oubliez pas de renommer la machine (`node2.lan1.tp1` devient `dhcp.lan1.tp1`)
- changez son adresse IP en `10.1.1.253`
- setup du serveur DHCP
  - commande d'installation du paquet
  - fichier de conf
  - service actif

☀️ **Sur `node1.lan1.tp1`**

- demandez une IP au serveur DHCP
- prouvez que vous avez bien récupéré une IP *via* le DHCP
- prouvez que vous avez bien récupéré l'IP de la passerelle
- prouvez que vous pouvez `ping node1.lan2.tp1`

## 2. Web web web

Un petit serveur web ? Pour la route ?

On recycle ici, toujours dans un soucis d'économie de ressources, la machine `node2.lan2.tp1` qui devient `web.lan2.tp1`. On va y monter un serveur Web qui mettra à disposition un site web tout nul.

---

La conf du serveur web :

- ce sera notre vieil ami NGINX
- il écoutera sur le port 80, port standard pour du trafic HTTP
- la racine web doit se trouver dans `/var/www/site_nul/`
  - vous y créerez un fichier `/var/www/site_nul/index.html` avec le contenu de votre choix
- vous ajouterez dans la conf NGINX **un fichier dédié** pour servir le site web nul qui se trouve dans `/var/www/site_nul/`
  - écoute sur le port 80
  - répond au nom `site_nul.tp1`
  - sert le dossier `/var/www/site_nul/`
- n'oubliez pas d'ouvrir le port dans le firewall 🌼

---

☀️ **Sur `web.lan2.tp1`**

- n'oubliez pas de renommer la machine (`node2.lan2.tp1` devient `web.lan2.tp1`)
- setup du service Web
  - installation de NGINX
  - gestion de la racine web `/var/www/site_nul/`
  - configuration NGINX
  - service actif
  - ouverture du port firewall
- prouvez qu'il y a un programme NGINX qui tourne derrière le port 80 de la machine (commande `ss`)
- prouvez que le firewall est bien configuré

☀️ **Sur `node1.lan1.tp1`**

- éditez le fichier `hosts` pour que `site_nul.tp1` pointe vers l'IP de `web.lan2.tp1`
- visitez le site nul avec une commande `curl` et en utilisant le nom `site_nul.tp1`

![That's all folks](./img/thatsall.jpg)