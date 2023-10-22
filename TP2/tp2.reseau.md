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
| `node1.lan1.tp2` | `10.1.1.11`        | x                  |
| `node2.lan1.tp2` | `10.1.1.12`        | x                  |
| `node1.lan2.tp2` | x                  | `10.1.2.11`        |
| `node2.lan2.tp2` | x                  | `10.1.2.12`        |
| `router.tp2`     | `10.1.1.254`       | `10.1.2.254`       |

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

- `node1.lan1.tp2`
  - doit pouvoir ping `node2.lan1.tp2`
  - doit aussi pouvoir ping `router.tp2` (il a deux IPs ce `router.tp2`, `node1.lan1.tp2` ne peut ping que celle qui est dans son réseau : `10.1.1.254`)
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

☀️ Sur **`node1.lan1.tp2`**

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
- prouvez avec un `traceroute` que le paquet passe bien par `router.tp2`
```bash
[manon@node1lan1 ~]$ traceroute 10.1.2.12
traceroute to 10.1.2.12 (10.1.2.12), 30 hops max, 60 byte packets
 1  10.1.1.254 (10.1.1.254)  5.589 ms  5.378 ms  5.070 ms
 2  10.1.2.12 (10.1.2.12)  4.844 ms !X  4.417 ms !X  4.104 ms !X
 ```

# II. Interlude accès internet

**On va donner accès internet à tout le monde.** Le routeur aura un accès internet, et permettra à tout le monde d'y accéder : il sera la passerelle par défaut des membres du LAN1 et des membres du LAN2.

**Ajoutez une carte NAT au routeur pour qu'il ait un accès internet.**

☀️ **Sur `router.tp2`**

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
- dans le compte-rendu, mettez-moi que la conf des points précédents sur `node2.lan1.tp2`
- prouvez que `node2.lan1.tp2` a un accès internet :
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

Dans cette partie, on va **monter quelques services orientés réseau** au sein de la topologie, afin de la rendre un peu utile que diable. Des machines qui se `ping` c'est rigolo mais ça sert à rien, des machines qui font des trucs c'est mieux.

## 1. DHCP

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

- n'oubliez pas de renommer la machine (`node2.lan1.tp2` devient `dhcp.lan1.tp2`)
```bash
[manon@dhcp ~]$ hostname
dhcp.lan1.tp2.lab.ingesup
```

- changez son adresse IP en `10.1.1.253`
```bash
[manon@dhcp ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:3e:f9:97 brd ff:ff:ff:ff:ff:ff
    inet 10.1.1.253/24 brd 10.1.1.255 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe3e:f997/64 scope link
       valid_lft forever preferred_lft forever
```
- setup du serveur DHCP
  - commande d'installation du paquet
  - fichier de conf
  - service actif

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
subnet 10.1.1.0 netmask 255.255.255.0 {
    # specify the range of lease IP address
    range dynamic-bootp 10.1.1.100 10.1.1.200;
    # specify broadcast address
    option broadcast-address 10.1.1.255;
    # specify gateway
    option routers 10.1.1.254;
  }

[manon@dhcp ~]$ sudo systemctl enable --now dhcpd
Created symlink /etc/systemd/system/multi-user.target.wants/dhcpd.service → /usr/lib/systemd/system/dhcpd.service.

[manon@dhcp ~]$ sudo firewall-cmd --add-service=dhcp
[sudo] password for manon:
success

[manon@dhcp ~]$ sudo firewall-cmd --runtime-to-permanent
success

[manon@dhcp ~]$ systemctl status dhcpd
● dhcpd.service - DHCPv4 Server Daemon
     Loaded: loaded (/usr/lib/systemd/system/dhcpd.service; enabled; preset: disabled)
     Active: active (running) since Sat 2023-10-21 10:05:48 CEST; 56s ago
       Docs: man:dhcpd(8)
             man:dhcpd.conf(5)
   Main PID: 1667 (dhcpd)
     Status: "Dispatching packets..."
      Tasks: 1 (limit: 11052)
     Memory: 5.2M
        CPU: 10ms
     CGroup: /system.slice/dhcpd.service
             └─1667 /usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd.conf -user dhcpd -group dhcpd --no-pid

Oct 21 10:05:48 dhcp.lan1.tp2.lab.ingesup dhcpd[1667]: Config file: /etc/dhcp/dhcpd.conf
Oct 21 10:05:48 dhcp.lan1.tp2.lab.ingesup dhcpd[1667]: Database file: /var/lib/dhcpd/dhcpd.leases
Oct 21 10:05:48 dhcp.lan1.tp2.lab.ingesup dhcpd[1667]: PID file: /var/run/dhcpd.pid
Oct 21 10:05:48 dhcp.lan1.tp2.lab.ingesup dhcpd[1667]: Source compiled to use binary-leases
Oct 21 10:05:48 dhcp.lan1.tp2.lab.ingesup dhcpd[1667]: Wrote 0 leases to leases file.
Oct 21 10:05:48 dhcp.lan1.tp2.lab.ingesup dhcpd[1667]: Listening on LPF/enp0s8/08:00:27:3e:f9:97/10.1.1.0/24
Oct 21 10:05:48 dhcp.lan1.tp2.lab.ingesup dhcpd[1667]: Sending on   LPF/enp0s8/08:00:27:3e:f9:97/10.1.1.0/24
Oct 21 10:05:48 dhcp.lan1.tp2.lab.ingesup dhcpd[1667]: Sending on   Socket/fallback/fallback-net
Oct 21 10:05:48 dhcp.lan1.tp2.lab.ingesup dhcpd[1667]: Server starting service.
Oct 21 10:05:48 dhcp.lan1.tp2.lab.ingesup systemd[1]: Started DHCPv4 Server Daemon.
```

☀️ **Sur `node1.lan1.tp2`**

```bash
[manon@node1lan1 ~]$ cat /etc/sysconfig/network-scripts/ifcfg-enp0s8
DEVICE=enp0s8

BOOTPROTO=dhcp
ONBOOT=yes
```

- demandez une IP au serveur DHCP
```bash
[manon@node1lan1 ~]$ ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=55 time=13.9 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=55 time=13.1 ms
^C
--- 1.1.1.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 13.072/13.478/13.885/0.406 ms
[manon@node1lan1 ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:4e:6b:1a brd ff:ff:ff:ff:ff:ff
    inet 10.1.1.100/24 brd 10.1.1.255 scope global dynamic noprefixroute enp0s8
       valid_lft 454sec preferred_lft 454sec
    inet6 fe80::a00:27ff:fe4e:6b1a/64 scope link
       valid_lft forever preferred_lft forever
```

- prouvez que vous avez bien récupéré une IP *via* le DHCP
- prouvez que vous avez bien récupéré l'IP de la passerelle
```bash
[manon@node1lan1 ~]$ ip route
default via 10.1.1.254 dev enp0s8 proto static metric 100
default via 10.1.1.254 dev enp0s8 proto dhcp src 10.1.1.100 metric 100
10.1.1.0/24 dev enp0s8 proto kernel scope link src 10.1.1.100 metric 100
10.1.2.0/24 via 10.1.1.254 dev enp0s8 proto static metric 100
```

- prouvez que vous pouvez `ping node1.lan2.tp1`
```bash
[manon@node1lan1 ~]$ ping 10.1.2.11
PING 10.1.2.11 (10.1.2.11) 56(84) bytes of data.
64 bytes from 10.1.2.11: icmp_seq=1 ttl=63 time=1.77 ms
64 bytes from 10.1.2.11: icmp_seq=2 ttl=63 time=1.77 ms
^C
--- 10.1.2.11 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1003ms
rtt min/avg/max/mdev = 1.772/1.772/1.773/0.000 ms
```

## 2. Web web web

Un petit serveur web ? Pour la route ?

---

La conf du serveur web :

- ce sera notre vieil ami NGINX
- il écoutera sur le port 80, port standard pour du trafic HTTP
- la racine web doit se trouver dans `/var/www/site_nul/`
  - vous y créerez un fichier `/var/www/site_nul/index.html` avec le contenu de votre choix
- vous ajouterez dans la conf NGINX **un fichier dédié** pour servir le site web nul qui se trouve dans `/var/www/site_nul/`
  - écoute sur le port 80
  - répond au nom `site_nul.tp2`
  - sert le dossier `/var/www/site_nul/`
- n'oubliez pas d'ouvrir le port dans le firewall 🌼

---

☀️ **Sur `web.lan2.tp2`**

- n'oubliez pas de renommer la machine (`node2.lan2.tp2` devient `web.lan2.tp2`)
```bash
[manon@web ~]$ hostname
web.lan2.tp1.lab.ingesup
```
- setup du service Web
  - installation de NGINX
  ```bash
  [manon@web ~]$ sudo dnf install nginx
  [manon@web ~]$ sudo systemctl enable nginx
  Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service → /usr/lib/systemd/system/nginx.service.
  [manon@web ~]$ sudo systemctl start nginx
  ```
  - gestion de la racine web `/var/www/site_nul/`
   ```bash
   [manon@web ~]$ sudo mkdir -p /var/www/site_nul/html
   [sudo] password for manon:
   [manon@web ~]$ sudo chown -R manon:manon /var/www/site_nul/html
   [sudo] password for manon:
   [manon@web ~]$ vim /var/www/site_nul/html/index.html
   [manon@web ~]$ cat /etc/nginx/conf.d/site_nul.conf
   server {
           listen 80;
           listen [::]:80;

           root /var/www/site_nul/html;
           index index.html index.htm index.nginx-debian.html;

           server_name site_nul www.site_nul;

           location / {
                   try_files $uri $uri/ =404;
           }
   }
   [manon@web ~]$ sudo nginx -t
   nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
   nginx: configuration file /etc/nginx/nginx.conf test is successful
   [manon@web ~]$ sudo systemctl restart nginx
   [manon@web ~]$ sudo chcon -vR system_u:object_r:httpd_sys_content_t:s0 /var/www/site_nul/
   changing security context of '/var/www/site_nul/html/index.html'
   changing security context of '/var/www/site_nul/html'
   changing security context of '/var/www/site_nul/'
   ```
  - configuration NGINX

  - service actif
  ```bash
  [manon@web ~]$   systemctl status nginx
  ● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: disabled)
     Active: active (running) since Sat 2023-10-21 10:53:22 CEST; 2min 15s ago
    Process: 1458 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 1459 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 1460 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 1461 (nginx)
      Tasks: 2 (limit: 11052)
     Memory: 1.9M
        CPU: 18ms
     CGroup: /system.slice/nginx.service
             ├─1461 "nginx: master process /usr/sbin/nginx"
             └─1462 "nginx: worker process"

  Oct 21 10:53:22 web.lan2.tp2.lab.ingesup systemd[1]: Starting The   nginx HTTP and reverse proxy server...
  Oct 21 10:53:22 web.lan2.tp2.lab.ingesup nginx[1459]: nginx: the  configuration file /etc/nginx/nginx.conf syntax is ok
  Oct 21 10:53:22 web.lan2.tp2.lab.ingesup nginx[1459]: nginx:  configuration file /etc/nginx/nginx.conf test is successful
  Oct 21 10:53:22 web.lan2.tp2.lab.ingesup systemd[1]: Started The  nginx HTTP and reverse proxy server.
  ```
  - ouverture du port firewall
  ```bash
  [manon@web ~]$ sudo firewall-cmd --add-port=80/tcp --permanent
  success
  [manon@web ~]$ sudo firewall-cmd --reload
  success
  ```
  - prouvez qu'il y a un programme NGINX qui tourne derrière le port  80 de la machine (commande `ss`)
  ```bash
  [manon@web ~]$ sudo ss -lutnp | grep nginx
  [sudo] password for manon:
  tcp   LISTEN 0      511          0.0.0.0:80        0.0.0.0:*      users:(("nginx",pid=1462,fd=6),("nginx",pid=1461,fd=6))
  tcp   LISTEN 0      511             [::]:80           [::]:*      users:(("nginx",pid=1462,fd=7),("nginx",pid=1461,fd=7))
  ```
  - prouvez que le firewall est bien configuré
  ```bash
  [manon@web ~]$ sudo firewall-cmd --list-all
  public
  target: default
  icmp-block-inversion: no
  interfaces:
  sources:
  services: cockpit dhcpv6-client ssh
  ports: 80/tcp
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
  [manon@web ~]$ sudo firewall-cmd --reload
  success
  ```

☀️ **Sur `node1.lan1.tp2`**

- éditez le fichier `hosts` pour que `site_nul.tp2` pointe vers l'IP de `web.lan2.tp2`
```bash
[manon@node1lan1 ~]$ cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
10.1.2.12   site_nul.tp2
```
- visitez le site nul avec une commande `curl` et en utilisant le nom `site_nul.tp2`
```bash
[manon@node1lan1 ~]$ curl site_nul.tp2
<!DOCTYPE html>
<html>
<head>
<title>Site nul</title>
</head>
<body>

<h1>Welcome on my site nul</h1>
<p>Meow meow</p>

</body>
</html>
```