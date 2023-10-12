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

- l'adresse IP du serveur auquel vous êtes connectés pour regarder la vidéo
- le port du serveur auquel vous êtes connectés
- le port que votre PC a ouvert en local pour se connecter au port du serveur distant

---

☀️ **Requêtes DNS**

Déterminer...

- à quelle adresse IP correspond le nom de domaine `www.ynov.com`

> Ca s'appelle faire un "lookup DNS".

- à quel nom de domaine correspond l'IP `174.43.238.89`

> Ca s'appelle faire un "reverse lookup DNS".

---

☀️ **Hop hop hop**

Déterminer...

- par combien de machines vos paquets passent quand vous essayez de joindre `www.ynov.com`

---

☀️ **IP publique**

Déterminer...

- l'adresse IP publique de la passerelle du réseau (le routeur d'YNOV donc si vous êtes dans les locaux d'YNOV quand vous faites le TP)

---

☀️ **Scan réseau**

Déterminer...

- combien il y a de machines dans le LAN auquel vous êtes connectés

> Allez-y mollo, on va vite flood le réseau sinon. :)

![Stop it](./img/stop.png)

# III. Le requin

Faites chauffer Wireshark. Pour chaque point, je veux que vous me livrez une capture Wireshark, format `.pcap` donc.

Faites *clean* 🧹, vous êtes des grands now :

- livrez moi des captures réseau avec uniquement ce que je demande et pas 40000 autres paquets autour
  - vous pouvez sélectionner seulement certains paquets quand vous enregistrez la capture dans Wireshark
- stockez les fichiers `.pcap` dans le dépôt git et côté rendu Markdown, vous me faites un lien vers le fichier, c'est cette syntaxe :

```markdown
[Lien vers capture ARP](./captures/arp.pcap)
```

---

☀️ **Capture ARP**

- 📁 fichier `arp.pcap`
- capturez un échange ARP entre votre PC et la passerelle du réseau

> Si vous utilisez un filtre Wireshark pour mieux voir ce trafic, précisez-le moi dans le compte-rendu.

---

☀️ **Capture DNS**

- 📁 fichier `dns.pcap`
- capturez une requête DNS vers le domaine de votre choix et la réponse
- vous effectuerez la requête DNS en ligne de commande

> Si vous utilisez un filtre Wireshark pour mieux voir ce trafic, précisez-le moi dans le compte-rendu.

---

☀️ **Capture TCP**

- 📁 fichier `tcp.pcap`
- effectuez une connexion qui sollicite le protocole TCP
- je veux voir dans la capture :
  - un 3-way handshake
  - un peu de trafic
  - la fin de la connexion TCP

> Si vous utilisez un filtre Wireshark pour mieux voir ce trafic, précisez-le moi dans le compte-rendu.

---

![Packet sniffer](img/wireshark.jpg)

> *Je sais que je vous l'ai déjà servi l'an dernier lui, mais j'aime trop ce meme hihi 🐈*