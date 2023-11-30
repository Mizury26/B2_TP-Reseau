# TP5 : Intégration

Dans les deux TPs précédents, vous avez joué le rôle d'admin réseau, on y reviendra sûrement par la suite. Maintenant, c'est au tour de l'admin système qui est en vous de s'exprimer.

> Dans ce TP on va faire un exercice un peu spécial. Ce TP durera plusieurs séances, même si vous l'avez déjà fini...

**L'idée est simple : vouis allez réellement héberger une application serveur, développée maison, au sein du réseau de l'école.**

> Pour que l'exercice soit chouette, ne posez pas trop de questions aux dév et aux sécus (si tu dis rien aux dévs ce sera définitivement bien plus rigolo).

Dans les grandes lignes, vous allez :

- **héberger une application**
  - dév en Python
  - ne respecte peut-être pas certains bonnes pratiques
- se démerder pour **l'installer**
  - j'vous guide bien sûr
  - mais l'idée c'est de se retrouver avec une app inconnue sur les bras : c'est tout le job !
- lancer et tester l'app
  - on vérifie que ça explose pas
  - et qu'on peut se connecter avec un client pour profiter de l'app
- **monitorer** l'app
  - monitoring du port utilisé par l'app, s'il tombe, on saura
  - ça permettra de la remonter si nécessaire
- une fois qu'elle fonctionne, vous l'**hébergerez au sein du réseau de l'école**
  - dans une VM, on est pas des animaux
  - avec une interface "Bridge" pour que la VM soit connectée au réseau physique

# Sommaire

- [TP5 : Intégration](#tp5--intégration)
- [Sommaire](#sommaire)
- [0. Setup](#0-setup)
  - [I. Tester](#i-tester)
  - [II. Intégrer](#ii-intégrer)
  - [III. Héberger](#iii-héberger)

# 0. Setup

Pas de GNS pour ce TP, juste votre VBox ça l'fait. On va faire qu'une VM `hosting.tp5.b1`, un Rocky Linux.

La ptite **checklist** pour la VM `hosting.tp5.b1` de ce TP :

- [ ] carte réseau host-only avec IP statique
- [ ] une carte NAT, pour avoir un accès internet
- [ ] connexion SSH fonctionnelle avec une paire de clé (pas de password)
- [ ] firewall actif
- [ ] SELinux désactivé
- [ ] hostname défini

## I. Tester

Dans cette partie, on va déjà essayer de lancer l'application vitefé, voir si elle fonctionne à peu près correctement.  
**Juste : eske sa march enfet ?**

Tout se fait depuis la VM `hosting.tp5.b1`.

🌞 **Récupérer l'application dans la VM `hosting.tp5.b1`**

- je vous file le lien de l'app en cours
```bash
[manon@hosting python_App]$ ls
client.py  server.py
```

🌞 **Essayer de lancer l'app**

- bah juste lance le !
- c'est le **serveur** qu'il faut lancer
- il faut :
  - déterminer dans quel langage est codé l'application : **python**
  - éventuellement lire le code vitefé, mais c'est pas le but
  - juste lancer avec une commande adaptée !
  ```bash
  [manon@hosting python_App]$ sudo python server.py
    13337
    Le serveur tourne sur 10.5.1.2:13337
  ```
- une commande `ss` qui me montre que le programme écoute derrière IP:port
```bash
[manon@hosting python_App]$ ss -ltn
State             Recv-Q            Send-Q                       Local Address:Port                         Peer Address:Port            Process
LISTEN            0                 128                                0.0.0.0:22                                0.0.0.0:*
LISTEN            0                 1                                127.0.0.1:13337                             0.0.0.0:*
```

> Si des erreurs apparaissent au lancement du serveur, il faudra les traiter !

🌞 **Tester l'app depuis `hosting.tp5.b1`**

- maintenant, on lance le **client**
```bash
[manon@hosting python_App]$ sudo python client.py
Veuillez saisir une opération arithmétique : 2+3
```
```bash
[manon@hosting python_App]$ sudo python server.py
13337
Le serveur tourne sur 127.0.0.1:13337
Exception in thread Thread-1:
Traceback (most recent call last):
  File "/usr/lib64/python3.9/threading.py", line 980, in _bootstrap_inner
    self.run()
  File "/usr/lib64/python3.9/threading.py", line 917, in run
    self._target(*self._args, **self._kwargs)
  File "/home/manon/python_App/server.py", line 95, in check_last_client_connection
    if time.time() - last_client_time > 60:
NameError: name 'last_client_time' is not defined
Un client ('127.0.0.1', 40658) s'est connecté.
Le client ('127.0.0.1', 40658) a envoyé 2+3
Réponse envoyée au client ('127.0.0.1', 40658) : 5.
```


➜ **Note :**

- le client va essayer spontanément de se connecter à une adresse IP spécifique (celle choisie par le dév pour ses tests) et un port spécifique
- **il sera peut-être nécessaire d'éditer vitefé le code** pour que le client se connecte à une autre IP
- par exemple `127.0.0.1` pour qu'il se connecte au serveur qui tourne en local sur la machine

> Si des erreurs apparaissent au lancement du client, il faudra les traiter !

➜ Vous passez à la suite que si vous arrivez à lancer le serveur, que le client se connecte, et que vous pouvez utiliser l'app ! (et quelle app !)

## II. Intégrer

Ce qu'on appelle l'intégration d'une application, c'est le fait de l'installer mais aussi de l'adapter aux règles et bonnes pratiques d'une infrastructure donnée.

Il existe en effet des bonnes pratiques partagées par à peu près tous les admins, mais aussi des règles spécifiques à un environnement ou un autres. On peut par exemple imaginer des règles de sécu plus restrictives pour une infrastructure qui gère des données médicales.

Ici on va se cantonner à des réflexes assez élémentaires :

- on va **créer un *service* systemd**
  - pour gérer plus facilement le serveur
  - le gérer de façon unifiée, comme n'importe quel autre service de la machine
  - proposer des politiques de restart par exemple
- et **monitoring du port TCP** sur lequel le serveur écoute
  - on monitore pour avoir des infos au moins si le service tombe
  - on met en place cette surveillance avec mon copain Netdata

## Sommaire

- [II. Intégrer](#ii-intégrer)
  - [Sommaire](#sommaire)
  - [1. Environnement](#1-environnement)
  - [2. systemd service](#2-systemd-service)
    - [B. Service basique](#b-service-basique)
    - [C. Amélioration du service](#c-amélioration-du-service)
  - [3. Monitoring](#3-monitoring)

## 1. Environnement

Petite section pour préparer un environnement correct. On va faire le strict minimum.

C'est une application installée à la main, on va la positionner un peu à l'arrache dans `/opt`.

🌞 **Créer un dossier /opt/calculatrice**

- il doit contenir le code de l'application
```bash
[manon@hosting calculatrice]$ sudo mv /home/manon/python_App/server.py /opt/calculatrice/
sudo mv /home/manon/python_App/bs_server.py /opt/calculatrice/
[manon@hosting calculatrice]$ ls
server.py
```

## 2. systemd service

Dans cette section, on va créer un fichier `calculatrice.service` qui nous permettra de saisir des commandes comme :

```bash
$ sudo systemctl start calculatrice
$ sudo systemctl enable calculatrice

$ sudo journalctl -xe -u calculatrice
```

### B. Service basique

🌞 **Créer le fichier `/etc/systemd/system/calculatrice.service`**

- vous devrez apposer des permissions correctes sur ce fichier
  - c'est quoi "correctes" ?
  - go faire un `ls -al` dans le dossier pour voir les permissions des autres fichiers `.service` déjà présents et faire pareil
- un contenu minimal serait :

```bash
$ sudo cat /etc/systemd/system/calculatrice.service
[Unit]
Description=Super calculatrice réseau

[Service]
ExecStart=/usr/bin/python /opt/calculatrice/bs_server.py

[Install]
WantedBy=multi-user.target
```

> On préfère utiliser le chemin absolu vers `python` plutôt que juste écrire la commande. Cela permet d'éviter des soucis s'il existe plusieurs version de Python installée par exemple. Sur certaines versions de systemd, ça marche carrément pas si on précise pas le chemin absolu.

➜ **Il est nécessaire** de taper la commande `systemctl daemon-reload` dès qu'on modifie un fichier `.service` pour indiquer à systemd de relire les fichiers et adopter la nouvelle configuration.

```bash
[manon@hosting system]$ sudo touch calculatrice.service

[manon@hosting system]$ sudo chmod 755 calculatrice.service

[manon@hosting system]$ cat calculatrice.service
[Unit]
Description=Super calculatrice réseau

[Service]
ExecStart=/usr/bin/python /opt/calculatrice/bs_server.py

[Install]
WantedBy=multi-user.target

[manon@hosting system]$ sudo systemctl daemon-reload
```

🌞 **Démarrer le service**

- avec une commande `sudo systemctl start calculatrice`
- prouvez que...
  - le service est actif avec un `systemctl status`
  - le service tourne derrière un port donné avec un `ss`
  - c'est fonctionnel : vous pouvez utiliser l'app en lançant le client

```bash
[manon@hosting system]$ sudo systemctl start calculatrice.service

[manon@hosting system]$ sudo systemctl status calculatrice.service
● calculatrice.service - Super calculatrice réseau
     Loaded: loaded (/etc/systemd/system/calculatrice.service; disabled; preset: disabled)
     Active: active (running) since Tue 2023-11-28 11:11:53 CET; 8s ago
   Main PID: 1954 (python)
      Tasks: 2 (limit: 11039)
     Memory: 5.5M
        CPU: 66ms
     CGroup: /system.slice/calculatrice.service
             └─1954 /usr/bin/python /opt/calculatrice/bs_server.py

Nov 28 11:11:53 hosting.tp5.b1 systemd[1]: Started Super calculatrice réseau.
Nov 28 11:11:53 hosting.tp5.b1 python[1954]: Le serveur tourne sur 127.0.0.1:13337
[manon@hosting system]$ ss -ltn
State     Recv-Q    Send-Q       Local Address:Port        Peer Address:Port    Process
LISTEN    0         128                0.0.0.0:22               0.0.0.0:*
LISTEN    0         1                127.0.0.1:13337            0.0.0.0:*
LISTEN    0         128                   [::]:22                  [::]:*

[manon@hosting python_App]$ sudo python client.py
[sudo] password for manon:
Veuillez saisir une opération arithmétique : 3+3
```


### C. Amélioration du service

➜ **Redémarrage automatique**

- si un jour le service est down, pour n'importe quelle raison, il serait bon qu'il redémarre automatiquement
- c'est une des features de systemd, on va configurer ça !

🌞 **Configurer une politique de redémarrage** dans le fichier `calculatrice.service`

- ajoutez :
  - une clause `Restart=`
    - qui permet d'indiquer quand restart
    - vous ferez en sorte qu'il restart quand il y a une erreur, n'importe laquelle
    - à mettre dans la section `[Service]` du fichier
  - une clause `RestartSec=`
    - permet d'indiquer au bout de combien de secondes le service sera relancé
    - on part pour 30 secondes
    - n'hésitez pas à réduire pour vos tests au début, mais je veux 30 sec dans le compte-rendu :
    - à mettre dans la section `[Service]` du fichier
- un `cat` du fichier pour le compte-rendu

> [Référez-vous à cette page de la doc officielle de systemd](https://www.freedesktop.org/software/systemd/man/latest/systemd.service.html) pour + d'informations sur ces clauses. Notamment savoir quoi écrire pour `Restart=`.

```bash
[manon@hosting system]$ cat calculatrice.service
[Unit]
Description=Super calculatrice réseau

[Service]
ExecStart=/usr/bin/python /opt/calculatrice/bs_server.py
RestartSec=30
Restart=on-failure

[Install]
WantedBy=multi-user.target

[manon@hosting system]$ sudo systemctl daemon-reload
```

🌞 **Tester que la politique de redémarrage fonctionne**

- lancez le service, repérer son PID, et tuer le process avec un `kill -9 <PID>`

```bash
[manon@hosting ~]$ ps -el | grep python
4 S     0    1263       1  0  80   0 - 21710 -      ?        00:00:00 python

[manon@hosting ~]$ sudo kill -9 1263

[manon@hosting ~]$ sudo systemctl status calculatrice.service
● calculatrice.service - Super calculatrice réseau
     Loaded: loaded (/etc/systemd/system/calculatrice.service; disabled; preset: disabled)
     Active: activating (auto-restart) (Result: signal) since Tue 2023-11-28 11:43:17 CET; 2s ago
    Process: 1263 ExecStart=/usr/bin/python /opt/calculatrice/bs_server.py (code=killed, signal=KILL)
   Main PID: 1263 (code=killed, signal=KILL)
        CPU: 26ms

Nov 28 11:43:17 hosting.tp5.b1 systemd[1]: calculatrice.service: Main process exited, code=killed, status=9/KILL
Nov 28 11:43:17 hosting.tp5.b1 systemd[1]: calculatrice.service: Failed with result 'signal'.

```
- attendre et vérifier que le service redémarre
```bash
[manon@hosting ~]$ sudo systemctl status calculatrice.service
● calculatrice.service - Super calculatrice réseau
     Loaded: loaded (/etc/systemd/system/calculatrice.service; disabled; preset: disabled)
     Active: active (running) since Tue 2023-11-28 11:43:47 CET; 1min 20s ago
   Main PID: 1292 (python)
      Tasks: 1 (limit: 11039)
     Memory: 5.6M
        CPU: 27ms
     CGroup: /system.slice/calculatrice.service
             └─1292 /usr/bin/python /opt/calculatrice/bs_server.py

Nov 28 11:43:47 hosting.tp5.b1 python[1292]: Le serveur tourne sur 127.0.0.1:13337
Nov 28 11:44:47 hosting.tp5.b1 python[1292]: Exception in thread Thread-1:
Nov 28 11:44:47 hosting.tp5.b1 python[1292]: Traceback (most recent call last):
Nov 28 11:44:47 hosting.tp5.b1 python[1292]:   File "/usr/lib64/python3.9/threading.py", line 980, in _bootstrap_inner
Nov 28 11:44:47 hosting.tp5.b1 python[1292]:     self.run()
Nov 28 11:44:47 hosting.tp5.b1 python[1292]:   File "/usr/lib64/python3.9/threading.py", line 917, in run
Nov 28 11:44:47 hosting.tp5.b1 python[1292]:     self._target(*self._args, **self._kwargs)
Nov 28 11:44:47 hosting.tp5.b1 python[1292]:   File "/opt/calculatrice/bs_server.py", line 96, in check_last_client_connection
Nov 28 11:44:47 hosting.tp5.b1 python[1292]:     if time.time() - last_client_time > 60:
Nov 28 11:44:47 hosting.tp5.b1 python[1292]: NameError: name 'last_client_time' is not defined
```
---

➜ **Firewall !**

- pour le moment vous avez fait tous les tests localement
  - donc on a pas eu besoin de toucher au firewall
  - le firewall de Rocky Linux ne bloque aucun traffic depuis/vers `127.0.0.1`
- quand on va héberger l'app sur le réseau de l'école et accueillir des clients potentiels, il faudra ouvrir un port firewall

🌞 **Ouverture automatique du firewall** dans le fichier `calculatrice.service`

- ajouter une clause `ExecStartPre=` qui ouvre le port dans le firewall
  - faut juste passer en argument la commande à exécuter avant (pre) que le service démarre (start)
  - à mettre dans la section `[Service]` du fichier
- ajouter une clause `ExecStopPost=` qui ferme le port dans le firewall
  - faut juste passer en argument la commande à exécuter après (post) que le service ait quitté (stop)
  - c'est trigger quand on `systemctl stop` le service, mais aussi quand il ferme inopinément (comme notre `kill -9` juste avant)
  - à mettre dans la section `[Service]` du fichier
- un `cat` du fichier pour le compte-rendu

> Préférez utiliser le chemin absolu vers `firewall-cmd` plutôt que juste écrire la commande. Utilisez `which firewall-cmd`  dans votre terminal pour connaître le chemin vers la commande.

```bash
[manon@hosting system]$ cat calculatrice.service
[Unit]
Description=Super calculatrice réseau

[Service]
ExecStartPre=/usr/bin/firewall-cmd --add-port=13337/tcp --permanent
ExecStartPre=/usr/bin/firewall-cmd --reload
ExecStart=/usr/bin/python /opt/calculatrice/bs_server.py
RestartSec=30
Restart=on-failure
ExecStopPost=firewall-cmd --remove-port=13337/tcp --permanent
ExecStopPost=/usr/bin/firewall-cmd --reload

[Install]
WantedBy=multi-user.target

[manon@hosting system]$ sudo systemctl daemon-reload
```

🌞 **Vérifier l'ouverture automatique du firewall**

- lancer/stopper le service et constater avec un `firewall-cmd --list-all` que le port s'ouvre/se ferme bien automatiquement

```bash
[manon@hosting system]$ sudo systemctl start calculatrice.service

[manon@hosting system]$ sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources:
  services: cockpit dhcpv6-client ssh
  ports: 13337/tcp
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:

[manon@hosting system]$ sudo systemctl stop calculatrice.service

[manon@hosting system]$ sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources:
  services: cockpit dhcpv6-client ssh
  ports: 
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

- tester une connexion depuis **votre PC**
  - récupérer le client sur votre PC
  - le lancer en lui indiquant de se connecter à l'IP de la VM
  - accéder au service

```bash
[manon@testeur python_App]$ sudo python client.py
Veuillez saisir une opération arithmétique : 3-3
```

## 3. Monitoring

🌞 **Installer Netdata** sur `hosting.tp5.b1`

- en suivant la doc officielle
```bash
[manon@hosting ~]$ wget -O /tmp/netdata-kickstart.sh https://my-netdata.io/kickstart.sh && sh /tmp/netdata-kickstart.sh -y

[manon@hosting ~]$ sudo systemctl start netdata

[manon@hosting ~]$ sudo systemctl enable netdata

[manon@hosting ~]$ sudo firewall-cmd --permanent --add-port=19999/tcp
success

[manon@hosting ~]$ sudo firewall-cmd --reload
success

[manon@hosting ~]$ sudo systemctl status netdata
[sudo] password for manon:
Sorry, try again.
[sudo] password for manon:
● netdata.service - Real time performance monitoring
     Loaded: loaded (/usr/lib/systemd/system/netdata.service; enabled; preset: enabled)
     Active: active (running) since Thu 2023-11-30 10:27:13 CET; 10min ago
   Main PID: 2673 (netdata)
      Tasks: 79 (limit: 11039)
     Memory: 92.6M
        CPU: 8.579s
     CGroup: /system.slice/netdata.service
             ├─2673 /opt/netdata/bin/srv/netdata -P /run/netdata/netdata.pid -D
             ├─2676 /opt/netdata/bin/srv/netdata --special-spawn-server
             ├─2837 bash /opt/netdata/usr/libexec/netdata/plugins.d/tc-qos-helper.sh 1
             ├─2851 /opt/netdata/usr/libexec/netdata/plugins.d/apps.plugin 1
             ├─2855 /opt/netdata/usr/libexec/netdata/plugins.d/debugfs.plugin 1
             ├─2856 /opt/netdata/usr/libexec/netdata/plugins.d/ebpf.plugin 1
             ├─2868 /opt/netdata/usr/libexec/netdata/plugins.d/go.d.plugin 1
             └─2871 /opt/netdata/usr/libexec/netdata/plugins.d/nfacct.plugin 1

Nov 30 10:28:19 hosting.tp5.b1 netdata[2673]: ALERT '1m_sent_traffic_overflow' of instance 'net.enp0s9' on node 'hosting.tp5.b1', transitioned from UNINITI>
Nov 30 10:28:19 hosting.tp5.b1 netdata[2673]: ALERT '1m_received_packets_rate' of instance 'net_packets.enp0s9' on node 'hosting.tp5.b1', transitioned from>
Nov 30 10:28:19 hosting.tp5.b1 netdata[2673]: ALERT 'ml_1min_node_ar' of instance 'anomaly_detection.anomaly_rate_on_a59c6c56-8f62-11ee-b66c-080027ab2d6c' >
Nov 30 10:28:29 hosting.tp5.b1 netdata[2673]: ALERT '1m_ip_tcp_resets_sent' of instance 'ip.tcphandshake' on node 'hosting.tp5.b1', transitioned from UNINI>
```
- assurez-vous que le dashboard Web est fonctionnel une fois l'install terminée

```bash
[manon@hosting ~]$ curl 10.5.1.2:19999
<!doctype html><html><head><title>Netdata Agent Console</title><script>let pathsRegex = /\/(spaces|nodes|overview|alerts|dashboards|anomalies|events|cloud|v2)\/?.*/
[...]
```

🌞 **Configurer une sonde TCP**

- c'est à dire qu'on va demander à Netdata de faire une requête TCP vers un port
  - si le port répond, Netdata considère que le service est up
  - sinon, il considère que c'est down
- ça va nous permettre de suivre un peu en temps réel si notre service est accessible depuis le réseau
- [cette section de la doc](https://learn.netdata.cloud/docs/data-collection/synthetic-checks/tcp-endpoints) parle de comment faire, lisez et check les exemples

> Dans le monde réel, le serveur de monitoring qui fait ce genre de checks est souvent installé sur une autre machine. Comme ça on simule vraiment un accès par le réseau à l'application, pour savoir si elle est disponible.

```bash
[manon@hosting ~]$ cd /etc/netdata 2>/dev/null || cd /opt/netdata/etc/netdata

[manon@hosting netdata]$ sudo ./edit-config go.d/portcheck.conf
Editing '/opt/netdata/etc/netdata/go.d/portcheck.conf' ...

## All available configuration options, their descriptions and default values:
## https://github.com/netdata/go.d.plugin/tree/master/modules/portcheck

#update_every: 1
autodetection_retry: 1
#priority: 70000

jobs:
 - name: calc
   host: 10.0.4.15
   ports: [13337]
```
🌞 **Alerting Discord**

- vous connaissez la chanson : j'aimerai que vous récupériez des alertes automatiquement sur Discord
- [cette section de la doc qui en parle](https://learn.netdata.cloud/docs/alerting/notifications/agent-dispatched-notifications/discord)

```bash
#------------------------------------------------------------------------------
# discord (discord.com) global notification options

# multiple recipients can be given like this:
#                  "CHANNEL1 CHANNEL2 ..."

# enable/disable sending discord notifications
SEND_DISCORD="YES"

# Create a webhook by following the official documentation -
# https://support.discord.com/hc/en-us/articles/228383668-Intro-to-Webhooks
DISCORD_WEBHOOK_URL="https://discord.com/api/webhooks/1179726649802104893/gmS8jOzcMd9mfUsh7cSyUayDIH6DWrwFWtgXmHRAZGSdDX0Yb_usovEs7BLHQWP0FPQP"

# if a role's recipients are not configured, a notification will be send to
# this discord channel (empty = do not send a notification for unconfigured
# roles):
DEFAULT_RECIPIENT_DISCORD="alarms"


#------------------------------------------------------------------------------
```
- testez que vous recevez une alerte quand vous coupez le service, et que votre sonde TCP n'a plus de réponse

## III. Héberger

Dans cette étape on va **rendre disponible le service Calculatrice au sein du réseau de l'école** grâce à une feature de VBox : les interfaces *bridge*.

Une interface *bridge* permet à la VM d'avoir un accès au réseau physique de l'hôte. Autrement dit, avoir une IP dans le LAN physique de l'hôte, et agir comme si c'était une autre machine du réseau.

Dans notre cas, ça va permettre de rendre le service de calculette accessible sur le réseau de l'école.

---

On parle d'ouvrir la machine sur le réseau de l'école, donc on va aussi s'assurer que vous aurez pas de soucis :

- déjà c'est une VM donc même si un vilain hacker casse tout, il est confiné dans la VM
- on va fermer les ports inutilement ouverts
- on va s'assurer que chaque service est disponible au bon endroit
  - serveur SSH écoute derrière l'IP host-only uniquement
  - serveur Calculatrice écoute derrière l'IP bridge uniquement

## Sommaire

- [III. Héberger](#iii-héberger)
  - [Sommaire](#sommaire)
  - [1. Interface bridge](#1-interface-bridge)
  - [2. Firewall](#2-firewall)
  - [3. Serveur SSH](#3-serveur-ssh)
  - [4. Serveur Calculatrice](#4-serveur-calculatrice)

## 1. Interface bridge

➜ Conf VM `hosting.tp5.b1`

- éteindre la VM
- la carte NAT ne sera plus utile
- ajoutez lui une nouvelle carte réseau en bridge
- allumez la VM
- récupérez une IP en DHCP sur cette interface
  - vous devriez récup une IP dans la range du LAN de l'école
  - vous devriez avoir un accès internet

```bash
[manon@hosting ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:ab:2d:6c brd ff:ff:ff:ff:ff:ff
    inet 10.5.1.2/24 brd 10.5.1.255 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:feab:2d6c/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:69:4d:d3 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.4/24 brd 192.168.1.255 scope global dynamic noprefixroute enp0s9
       valid_lft 42578sec preferred_lft 42578sec
    inet6 2a01:e0a:5b9:c40:c117:1dfb:8f1c:c9bd/64 scope global dynamic noprefixroute
       valid_lft 86120sec preferred_lft 86120sec
    inet6 fe80::2491:4d9b:df:8756/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```
```bash
[manon@hosting ~]$ ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=58 time=11.6 ms
^C
```

## 2. Firewall

🌞 **Assurez-vous qu'aucun port est inutilement ouvert**
```bash
[manon@hosting ~]$ sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s8 enp0s9
  sources:
  services: cockpit dhcpv6-client ssh
  ports: 13337/tcp
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

## 3. Serveur SSH

🌞 **Conf serveur SSH**

- éditer la configuration du serveur SSH `/etc/ssh/sshd_config`
- le serveur doit être disponible uniquement sur la carte host-only (pas sur l'interface bridge)

```bash
[manon@hosting ssh]$ sudo cat sshd_config | grep ListenAddress
[sudo] password for manon:
ListenAddress 10.5.1.2
```
- n'oubliez pas de redémarrer le serveur pour que ça prenne effet

```bash
[manon@hosting ssh]$ sudo systemctl restart sshd
```

- prouvez avec un `ss` que ça a pris effet
```bash
[manon@hosting ssh]$ ss -ltn
State      Recv-Q     Send-Q          Local Address:Port            Peer Address:Port     Process
LISTEN     0          1                    10.5.1.2:13337                0.0.0.0:*
LISTEN     0          128                  10.5.1.2:22                   0.0.0.0:*
```

## 4. Serveur Calculatrice

🌞 **Conf serveur Calculatrice**

- bon les dévs n'ont pas eu la bonté de nous faire un fichier de conf
- donc go éditer le code pour que le serveur n'écoute que sur une seule IP : celle de l'interface bridge

-> Mise en place d'une redirection de port avec la carte Nat + ouverture du port

- n'oubliez pas de redémarrer le serveur pour que ça prenne effet

```bash
[manon@hosting netdata]$ sudo systemctl restart calculatrice.service
```
- prouvez avec un `ss` que le changement a bien pris effet

```bash
[manon@hosting netdata]$ ss -ltn
State   Recv-Q  Send-Q   Local Address:Port     Peer Address:Port  Process
LISTEN  0       128           10.5.1.2:22            0.0.0.0:*
LISTEN  0       4096         127.0.0.1:8125          0.0.0.0:*
LISTEN  0       1            10.0.4.15:13337         0.0.0.0:*
LISTEN  0       4096           0.0.0.0:19999         0.0.0.0:*
LISTEN  0       4096             [::1]:8125             [::]:*
LISTEN  0       4096              [::]:19999            [::]:*
```