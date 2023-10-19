# TP1 : Maîtrise réseau du poste


- [TP1 : Maîtrise réseau du poste](#tp1--maîtrise-réseau-du-poste)
- [I. Basics](#i-basics)
- [II. Go further](#ii-go-further)
- [III. Le requin](#iii-le-requin)

# I. Basics

☀️ **Carte réseau WiFi**

Déterminer...

- l'adresse MAC de votre carte WiFi
- l'adresse IP de votre carte WiFi
- le masque de sous-réseau du réseau LAN auquel vous êtes connectés en WiFi
  - en notation CIDR, par exemple `/16`
  - ET en notation décimale, par exemple `255.255.0.0`

```
Carte réseau sans fil Wi-Fi :

   Adresse physique . . . . . . . . . . . : B2-4A-7A-7A-8E-03
   Adresse IPv6 de liaison locale. . . . .: fe80::c46f:dd89:9319:e6a%6(préféré)
   Adresse IPv4. . . . . . . . . . . . . .: 192.168.43.239(préféré)
   Masque de sous-réseau. . . . . . . . . : 255.255.255.0

   Masque de sous-réseau (en CIDR) : 192.168.43.239/24
```

---

☀️ **Déso pas déso**

Pas besoin d'un terminal là, juste une feuille, ou votre tête, ou un tool qui calcule tout hihi. Déterminer...

- l'adresse de réseau du LAN auquel vous êtes connectés en WiFi : 192.168.43.0
- l'adresse de broadcast : 192.168.43.255
- le nombre d'adresses IP disponibles dans ce réseau : 254
---

☀️ **Hostname**

- déterminer le hostname de votre PC
```
PS C:\Users\Utilisateur\Documents\B2_info\reseau_b2> hostname
HP-ProBook-450
```

☀️ **Passerelle du réseau**

Déterminer...

- l'adresse IP de la passerelle du réseau
- l'adresse MAC de la passerelle du réseau

```
PS C:\Users\Utilisateur\Documents\B2_info\reseau_b2> ipconfig -all
Carte réseau sans fil Wi-Fi :

   Passerelle par défaut. . . . . . . . . : 192.168.43.1

PS C:\Users\Utilisateur\Documents\B2_info\reseau_b2> arp -a

Interface : 192.168.43.239 --- 0x6
  Adresse Internet      Adresse physique      Type
  192.168.43.1          0a-c5-e1-b2-52-53     dynamique
---
```

☀️ **Serveur DHCP et DNS**

Déterminer...

- l'adresse IP du serveur DHCP qui vous a filé une IP
- l'adresse IP du serveur DNS que vous utilisez quand vous allez sur internet

```
PS C:\Users\Utilisateur\Documents\B2_info\reseau_b2> ipconfig -all
Carte réseau sans fil Wi-Fi :

   Serveur DHCP . . . . . . . . . . . . . : 192.168.43.1
   Serveurs DNS. . .  . . . . . . . . . . : 192.168.43.1
```
---

☀️ **Table de routage**

Déterminer...

- dans votre table de routage, laquelle est la route par défaut
```
PS C:\Users\Utilisateur\Documents\B2_info\reseau_b2> netstat -r
IPv4 Table de routage
===========================================================================
Itinéraires actifs :
Destination réseau    Masque réseau  Adr. passerelle   Adr. interface Métrique
          0.0.0.0          0.0.0.0     192.168.43.1   192.168.43.239     50

```

---

# II. Go further

> Toujours tout en ligne de commande.

---

☀️ **Hosts ?**

- faites en sorte que pour votre PC, le nom `b2.hello.vous` corresponde à l'IP `1.1.1.1`
- prouvez avec un `ping b2.hello.vous` que ça ping bien `1.1.1.1`

```
PS C:\Windows\System32\drivers\etc> ping b2.hello.vous

Envoi d’une requête 'ping' sur b2.hello.vous [1.1.1.1] avec 32 octets de données :
Réponse de 1.1.1.1 : octets=32 temps=286 ms TTL=56
Réponse de 1.1.1.1 : octets=32 temps=21 ms TTL=56
Réponse de 1.1.1.1 : octets=32 temps=22 ms TTL=56

Statistiques Ping pour 1.1.1.1:
    Paquets : envoyés = 3, reçus = 3, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 21ms, Maximum = 286ms, Moyenne = 109ms
```

---

☀️ **Go mater une vidéo youtube et déterminer, pendant qu'elle tourne...**

- l'adresse IP du serveur auquel vous êtes connectés pour regarder la vidéo : 
91.68.245.78
- le port du serveur auquel vous êtes connectés : 443
- le port que votre PC a ouvert en local pour se connecter au port du serveur distant : 57363

[capture wireshark](/TP1/capture_youtube.pcap)

---

☀️ **Requêtes DNS**

Déterminer...

- à quelle adresse IP correspond le nom de domaine `www.ynov.com`
```
PS C:\Users\Utilisateur> nslookup www.ynov.com
Serveur :   dns.google
Address:  8.8.8.8

Réponse ne faisant pas autorité :
Nom :    www.ynov.com
Addresses:  2606:4700:20::681a:be9
          2606:4700:20::ac43:4ae2
          2606:4700:20::681a:ae9
          104.26.11.233
          172.67.74.226
          104.26.10.233

```

- à quel nom de domaine correspond l'IP `174.43.238.89`

```
PS C:\Users\Utilisateur> nslookup 174.43.238.89
Serveur :   dns.google
Address:  8.8.8.8

Nom :    89.sub-174-43-238.myvzw.com
Address:  174.43.238.89
```

---

☀️ **Hop hop hop**

Déterminer...

- par combien de machines vos paquets passent quand vous essayez de joindre `www.ynov.com`

```
PS C:\Users\Utilisateur> tracert www.ynov.com

Détermination de l’itinéraire vers www.ynov.com [104.26.11.233]
avec un maximum de 30 sauts :

  1     4 ms     3 ms     2 ms  10.33.79.254
  2     2 ms     6 ms     3 ms  145.117.7.195.rev.sfr.net [195.7.117.145]
  3     9 ms     5 ms     5 ms  237.195.79.86.rev.sfr.net [86.79.195.237]
  4     5 ms     3 ms     4 ms  196.224.65.86.rev.sfr.net [86.65.224.196]
  5    12 ms    12 ms    11 ms  12.148.6.194.rev.sfr.net [194.6.148.12]
  6    12 ms    12 ms    63 ms  12.148.6.194.rev.sfr.net [194.6.148.12]
  7    28 ms    15 ms    15 ms  141.101.67.48
  8    11 ms    11 ms    11 ms  172.71.124.4
  9    11 ms    10 ms    10 ms  104.26.11.233

Itinéraire déterminé.
```
---

☀️ **IP publique**

Déterminer...

- l'adresse IP publique de la passerelle du réseau (le routeur d'YNOV donc si vous êtes dans les locaux d'YNOV quand vous faites le TP)
```
PS C:\Users\Utilisateur> curl ifconfig.me


StatusCode        : 200
StatusDescription : OK
Content           : 195.7.117.146
```
---

☀️ **Scan réseau**

Déterminer...

- combien il y a de machines dans le LAN auquel vous êtes connectés

```
PS C:\Users\Utilisateur> nmap -sn 10.33.64.136/20
Nmap done: 4096 IP addresses (863 hosts up) scanned in 201.50 seconds
```

# III. Le requin


☀️ **Capture ARP**

- 📁 fichier `arp.pcap`
- capturez un échange ARP entre votre PC et la passerelle du réseau :
  - [arp](./arp.pcap) avec le filtre "arp"
---

☀️ **Capture DNS**

- 📁 fichier `dns.pcap`
- capturez une requête DNS vers le domaine de votre choix et la réponse
- vous effectuerez la requête DNS en ligne de commande
  - [dns](./dns.pcap) avec le filtre "dns"

---

☀️ **Capture TCP**

- 📁 fichier `tcp.pcap`
- effectuez une connexion qui sollicite le protocole TCP
  - [tcp](./tcp.pcapng) avec le filtre tcp

---