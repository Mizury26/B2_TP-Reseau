# TP8 INFRA : Make ur own

Dans ce TP on va encore monter d'un cran : on se met (presque) en situation réelle : vous êtes des admins réseau que je prends en presta pour ma boîte fictive.

J'ai actuellement un réseau foireux, et j'aimerai voir ce que me proposerait pour nouvelle architecture un prestataire externe.

> *"Presque" en situation réelle car on ne s'occupe ni de l'aspect pécunier (on va pas estimer la valeur de l'archi, ce ne sera pas un critère), ni de l'aspect physique (on suppose que les différents locaux de la boîte sont directement connectés, on omet la couche internet). De plus, pour la simplicité de l'exercice, on va considérer que chaque machine est physique. Dans un environnement réel, les serveurs sont des machines virtuelles.*


## Sommaire

- [TP8 INFRA : Make ur own](#tp8-infra--make-ur-own)
  - [Sommaire](#sommaire)
  - [I. Expression du besoin](#i-expression-du-besoin)
    - [1. Ptite intro](#1-ptite-intro)
    - [2. Présentation des équipes](#2-présentation-des-équipes)
    - [3. Equipements connectés](#3-equipements-connectés)
    - [4. Salle serveur](#4-salle-serveur)
    - [5. Exigences diverses](#5-exigences-diverses)
    - [6. Considérations spécifiques pour le TP](#6-considérations-spécifiques-pour-le-tp)
  - [II. Rendu attendu](#ii-rendu-attendu)

## I. Expression du besoin

### 1. Ptite intro

On est répartis sur deux sites physiques en France (ou pas) :

- 🍷 **le site "Meow Origins"**
  - locaux à Bordeaux
  - 1 seul bâtiment, 3 étages et un sous-sol
  - y'a une salle serveur au sous-sol
  - architecture type *3-tier architecture*
- 🚀 **le site "Meow and Beyond"**
  - nouveaux locaux !
  - locaux sur la Lune
  - 2 bâtiments, pas d'étage
    - reliés en direct par un câble, ils sont collés !
  - salle serveur dans le bâtiment 1
  - architecture type *router-on-a-stick*
- les deux sites
  - sont directement connectés
  - ont tous les deux un accès direct à internet

On a une volonté de garder la maîtrise sur notre infra, alors on héberge tout en interne.

On est paranos un peu alors il n'y a pas de WiFi dans nos locaux, et on fournit nous-mêmes les postes de travail à nos utilisateurs (les dévs et autres).

### 2. Présentation des équipes

➜ 🍷 **Site "Meow Origins"**

- équipe dév
  - 7 lead devs
  - 132 dévs
  - tous répartis dans deux big open-space étage 1 et étage 2
- équipe admin
  - 1 admin réseau
  - 1 admin sys
  - 1 responsable sécu
  - tous dans l'open-space étage 2
- direction
  - 1 PDG
    - un bureau pour lui étage 2
  - 5 secrétaires/agents d'accueil
    - un bureau dédié étage 1 : ils/elles sont deux
    - rez-de-chaussée  : les 3 restant(e)s
  - 2 agents RH
    - un bureau dédié étage 2

➜ 🚀 **Site "Meow and Beyond"**

- équipe dév
  - 2 lead devs
  - 12 dévs
  - tous dans un open-space dans le bâtiment 2
- équipe admin
  - 1 admin réseau
  - 1 admin sys
  - tous dans un bureau dans le bâtiment 1
- direction
  - 2 secrétaires/agents d'accueil
  - dans un bureau dans le bâtiment 1

### 3. Equipements connectés

➜ **Imprimantes**

- on a une imprimante réseau à chaque étage de chaque bâtiment

➜ **Caméras**

- 2 caméras à chaque étage de chaque bâtiment
- 1 caméra à l'entrée de chaque bâtiment

➜ **Télés**

- 2 télés à l'accueil de chaque bâtiment
- 1 télé à chaque étage hors rez-de-chaussée de chaque bâtiment

➜ **Téléphone IP**

- 1 téléphone IP par employé

### 4. Salle serveur

➜ 🍷 **Site "Meow Origins"**

- serveur DHCP
  - donne des IP à tous les réseaux de clients
  - pas les serveurs/routeurs, etc. (évidemment ! :D)
- serveur DNS
  - permet de résoudre les noms de TOUTES les machines des deux sites
  - notre domaine c'est `dev.meow`
    - par exemple notre serveur DNS c'est `dns.dev.meow`
- plateforme de production
- plateforme de tests
- dépôts git internes
- accès internet
- accès à l'autre site

➜ 🚀 **Site "Meow and Beyond"**

- serveur DHCP
  - donne des IP à tous les réseaux de clients
  - pas les serveurs/routeurs, etc. (évidemment ! :D)
- plateforme de tests
- accès internet
- accès à l'autre site

### 5. Exigences diverses

➜ **Plateforme de test**

- nous avons l'habitude de fournir aux dévs une plateforme de test
- c'est à dire un réseau qui héberge des machines dédiées aux tests des dévs
- ils peuvent se connecter à ces machines et lancer leur code
- ces machines sont (quasiment...) identiques aux machines de production
- actuellement les environnements de test comportent 30 machines
- un serveur de database est aussi présent en plus des 30 serveurs de test

➜ **Production**

- nous avons un réseau dédié qui héberge des serveurs de production
- il existe un serveur dédié à chaque application que nous développons
- en ce moment nous avons donc deux serveurs de production
- un serveur de database est aussi présent en plus, pour servir ces 2 serveurs de production

➜ **Dépôts git**

- on héberge nous-mêmes des dépôts git pour stocker le code produit en interne par nos développeurs
- on a actuellement 1 seul serveur Git hébergé sur le 🍷 **Site "Meow Origins"**

### 6. Considérations spécifiques pour le TP

➜ **Choix des OS**

- les routeurs
  - Cisco ou Rocky Linux
- les switches
  - Cisco
- serveur DHCP
  - Rocky Linux
- serveur DNS
  - Rocky Linux
- tout le reste est simulé
  - VPCS ou Rocky Linux

➜ **Les locaux**

- on considère que les deux sites sont connectés en direct avec un câble
- les deux sites disposent de leur propre accès internet
- les deux sites sont routés en direct, c'est à dire que des machines du site A peuvent ping des machines du site B
- en revanche, aucun réseau IP ni VLAN n'est partagé entre les deux sites
  - on simule une situation réelle où il y a internet entre les deux
  - pas de réseau IP dupliqué des deux côtés

## II. Rendu attendu

➜ Je vous recommande FORTEMENT de suivre la démarche suivante :

- faire un schéma réseau
  - me le soumettre
  - éventuellement l'ajuster en fonction de mes retours
- établir le tableau d'adressage IP/VLAN
- monter la topologie dans GNS
- configurer uniquement la partie L2/L3
  - c'est à dire les switches, les routeurs, accès à internet
- puis passer à la conf des serveurs Linux
  - DHCP et DNS notamment

🌞 **Rendu Markdown**

- comme d'hab quoi
- comporte tous les points qui suivent
  - soit des documents liés
  - soit directement du Markdown

🌞 **Schéma réseau**

![Topo](./img/topo_tp8.png)

🌞 **Tableaux d'adressage et VLAN**


#### 1. Tableau d'adressage

**- Site "Meow Origins" :**

| Machine - Réseau    | `10.10.0.0/23` | `10.20.0.0/28` | `10.30.0.0/27` | `10.40.0.0/24` | `10.50.0.0/28` | `10.60.0.0/28` | `10.70.0.0/28` | `10.80.0.0/23` | `10.90.0.0/28` | `10.8.1.0/30`  |
| ------------------- | -------------- | -------------- | -------------- | -------------- | -------------- | -------------- | -------------- | -------------- | -------------- | -------------- |
| `r1`                | `10.10.1.254`  | `10.20.0.14`   | `10.30.0.30`   | `10.40.0.254`  | `10.50.0.14`   | `10.60.0.14`   | `10.70.0.14`   | `10.80.1.254`  | `10.90.0.14`  | `10.8.1.1/30`  |
| `devlead1.ori.meo`  | `10.10.0.1`    | ❌            | ❌             | ❌             |  ❌           |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `devlead7.ori.meo`  | `10.10.0.7`    | ❌            | ❌             | ❌             |  ❌           |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `dev1.ori.meo`      | `10.10.0.8`    | ❌            | ❌             | ❌             |  ❌           |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `dev132.ori.meo`    | `10.10.0.139`  | ❌            | ❌             | ❌             |  ❌           |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `admres.ori.meo`    | ❌             | `10.20.0.1`   | ❌             | ❌             |  ❌           |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `admsys.ori.meo`    | ❌             | `10.20.0.2`   | ❌             | ❌             |  ❌           |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `admsecu.ori.meo`   | ❌             | `10.20.0.3`   | ❌             | ❌             |  ❌           |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `pdg.ori.meo`       | ❌             | ❌            | `10.30.0.1`    | ❌             |  ❌           |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `sec1.ori.meo`      | ❌             | ❌            | `10.30.0.2`    | ❌             |  ❌           |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `sec2.ori.meo`      | ❌             | ❌            | `10.30.0.3`    | ❌             |  ❌           |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `sec3.ori.meo`      | ❌             | ❌            | `10.30.0.4`    | ❌             |  ❌           |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `sec4.ori.meo`      | ❌             | ❌            | `10.30.0.5`    | ❌             |  ❌           |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `sec5.ori.meo`      | ❌             | ❌            | `10.30.0.6`    | ❌             |  ❌           |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `rh.1.ori.meo`      | ❌             | ❌            | `10.30.0.7`    | ❌             |  ❌           |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `rh.2.ori.meo`      | ❌             | ❌            | `10.30.0.8`    | ❌             |  ❌           |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `testserv1.ori.meo` | ❌             | ❌            | ❌            | `10.40.0.1`     |  ❌           |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `testserv30.ori.meo`| ❌             | ❌            | ❌            | `10.40.0.2`     |  ❌           |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `testdb.ori.meo`    | ❌             | ❌            | ❌            | `10.40.0.3`     |  ❌           |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `prodserv1.ori.meo` | ❌             | ❌            | ❌            | `10.40.0.4`     |  ❌           |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `prodserv2.ori.meo` | ❌             | ❌            | ❌            | `10.40.0.5`     |  ❌           |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `proddb.ori.meo`    | ❌             | ❌            | ❌            | `10.40.0.6`     |  ❌           |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `gitserv.ori.meo`   | ❌             | ❌            | ❌            | `10.40.0.7`     |  ❌           |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `imp1.ori.meo`      | ❌             | ❌            | ❌            | ❌             | `10.50.0.1`    |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `imp2.ori.meo`      | ❌             | ❌            | ❌            | ❌             | `10.50.0.2`    |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `imp3.ori.meo`      | ❌             | ❌            | ❌            | ❌             | `10.50.0.3`    |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `cam1.ori.meo`      | ❌             | ❌            | ❌            | ❌             | ❌             | `10.60.0.1`   | ❌             | ❌            | ❌             | ❌             |
| `cam2.ori.meo`      | ❌             | ❌            | ❌            | ❌             | ❌             | `10.60.0.2`   | ❌             | ❌            | ❌             | ❌             |
| `cam3.ori.meo`      | ❌             | ❌            | ❌            | ❌             | ❌             | `10.60.0.3`   | ❌             | ❌            | ❌             | ❌             |
| `cam4.ori.meo`      | ❌             | ❌            | ❌            | ❌             | ❌             | `10.60.0.4`   | ❌             | ❌            | ❌             | ❌             |
| `cam5.ori.meo`      | ❌             | ❌            | ❌            | ❌             | ❌             | `10.60.0.5`   | ❌             | ❌            | ❌             | ❌             |
| `tv1.ori.meo`       | ❌             | ❌            | ❌            | ❌             | ❌             | ❌            | `10.70.0.1`    | ❌            | ❌             | ❌             |
| `tv2.ori.meo`       | ❌             | ❌            | ❌            | ❌             | ❌             | ❌            | `10.70.0.2`    | ❌            | ❌             | ❌             |
| `tv3.ori.meo`       | ❌             | ❌            | ❌            | ❌             | ❌             | ❌            | `10.70.0.3`    | ❌            | ❌             | ❌             |
| `tv4.ori.meo`       | ❌             | ❌            | ❌            | ❌             | ❌             | ❌            | `10.70.0.4`    | ❌            | ❌             | ❌             |
| `tel1.ori.meo`      | ❌             | ❌            | ❌            | ❌             | ❌             | ❌            | ❌            | `10.80.0.1`    | ❌             | ❌             |
| `tel2.ori.meo`      | ❌             | ❌            | ❌            | ❌             | ❌             | ❌            | ❌            | `10.80.0.2`    | ❌             | ❌             |
| `tel3.ori.meo`      | ❌             | ❌            | ❌            | ❌             | ❌             | ❌            | ❌            | `10.80.0.3`    | ❌             | ❌             |
| `tel4.ori.meo`      | ❌             | ❌            | ❌            | ❌             | ❌             | ❌            | ❌            | `10.80.0.4`    | ❌             | ❌             |
| `tel5.ori.meo`      | ❌             | ❌            | ❌            | ❌             | ❌             | ❌            | ❌            | `10.80.0.5`    | ❌             | ❌             |
| `dns.ori.meow`      | ❌             | ❌            | ❌            | ❌             | ❌             | ❌            | ❌            | ❌            | `10.90.0.1`     | ❌             |
| `dhcp.ori.meo`      | ❌             | ❌            | ❌            | ❌             | ❌             | ❌            | ❌            | ❌            | `10.90.0.2`     | ❌             |
 
**- Site "Meow and Beyond" :**

| Machine - Réseau        | `10.15.0.0/24` | `10.25.0.0/28` | `10.35.0.0/27` | `10.45.0.0/24` | `10.55.0.0/28` | `10.65.0.0/28` | `10.75.0.0/28` | `10.85.0.0/24` | `10.95.0.0/28` | `10.8.1.0/30`  |
| -----------------       | -------------- | -------------- | -------------- | -------------- | -------------- | -------------- | -------------- | -------------- | -------------- | -------------- |
| `r2`                    | `10.15.0.254`  | `10.25.0.14`   | `10.35.0.30`   | `10.45.0.254`  | `10.55.0.14`   | `10.65.0.14`   | `10.75.0.14`   | `10.85.0.254`  | `10.95.0.14`   | `10.8.1.2/30`  |
| `devlead1.bey.meo`      | `10.15.0.1`    | ❌            | ❌             | ❌             |  ❌           |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `dev1.bey.meo`          | `10.15.0.3`    | ❌            | ❌             | ❌             |  ❌           |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `admres.bey.meo`        | ❌             | `10.25.0.1`   | ❌             | ❌             |  ❌           |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `admsys.bey.meo`        | ❌             | `10.25.0.2`   | ❌             | ❌             |  ❌           |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `sec1.bey.meo`          | ❌             | ❌            | `10.35.0.1`    | ❌             |  ❌           |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `sec2.bey.meo`          | ❌             | ❌            | `10.35.0.2`    | ❌             |  ❌           |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `testserv1.bey.meo`     | ❌             | ❌            | ❌            | `10.45.0.1`     |  ❌           |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `testserv30.bey.meo`    | ❌             | ❌            | ❌            | `10.45.0.2`     |  ❌           |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `testdb.bey.meo`        | ❌             | ❌            | ❌            | `10.45.0.3`     |  ❌           |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `imp1.bey.meo`          | ❌             | ❌            | ❌            | ❌             | `10.55.0.1`    |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `imp2.bey.meo`          | ❌             | ❌            | ❌            | ❌             | `10.55.0.2`    |  ❌           | ❌             | ❌            | ❌             | ❌             |
| `cam1.bey.meo`          | ❌             | ❌            | ❌            | ❌             | ❌             | `10.65.0.1`   | ❌             | ❌            | ❌             | ❌             |
| `cam2.bey.meo`          | ❌             | ❌            | ❌            | ❌             | ❌             | `10.65.0.2`   | ❌             | ❌            | ❌             | ❌             |
| `tv1.bey.meo`           | ❌             | ❌            | ❌            | ❌             | ❌             | ❌            | `10.75.0.1`    | ❌            | ❌             | ❌             |
| `tv2.bey.meo`           | ❌             | ❌            | ❌            | ❌             | ❌             | ❌            | `10.75.0.2`    | ❌            | ❌             | ❌             |
| `tv3.bey.meo`           | ❌             | ❌            | ❌            | ❌             | ❌             | ❌            | `10.75.0.3`    | ❌            | ❌             | ❌             |
| `tv4.bey.meo`           | ❌             | ❌            | ❌            | ❌             | ❌             | ❌            | `10.75.0.4`    | ❌            | ❌             | ❌             |
| `tel1.bey.meo`          | ❌             | ❌            | ❌            | ❌             | ❌             | ❌            | ❌            | `10.85.0.1`    | ❌             | ❌             |
| `tel2.bey.meo`          | ❌             | ❌            | ❌            | ❌             | ❌             | ❌            | ❌            | `10.85.0.2`    | ❌             | ❌             |
| `dhcp.bey.meo`          | ❌             | ❌            | ❌            | ❌             | ❌             | ❌            | ❌            | ❌            | `10.95.0.1`     | ❌             |


#### 2. Tableau des VLANs

- Association VLAN <> réseau IP 

**- Site "Meow Origins" :**

| VLAN              | VLAN 10 `dev1`    | VLAN 20 `adm1`    | VLAN 30 `dir1`    | VLAN 40 `serv1`   | VLAN 50 `imp1`    | VLAN 60 `cam1`    | VLAN 70 `tv1`     | VLAN 80 `tel1`    | VLAN 90 `service1`|
| ----------------- | ----------------- | ----------------- | ----------------- | ----------------- | ----------------- | ----------------- | ----------------- | ----------------- | ----------------- |
| Réseau IP associé | `10.10.0.0/23`    | `10.20.0.0/28`    | `10.30.0.0/27`    | `10.40.0.0/24`    | `10.50.0.0/28`    | `10.60.0.0/28`    | `10.70.0.0/28`    | `10.8.80.0/23`    | `10.90.0.0/28`    |


**- Site "Meow and Beyond" :**

| VLAN              | VLAN 15 `dev2`    | VLAN 25 `adm2`   | VLAN 35 `dir2`     | VLAN 45 `serv2`   | VLAN 55 `imp2`    | VLAN 65 `cam2`    | VLAN 75 `tv2`     | VLAN 85 `tel2`    | VLAN 95 `service2`|
| ----------------- | ----------------- | ---------------- | ------------------ | ----------------- | ----------------- | ----------------- | ----------------- | ----------------- | ----------------- |
| Réseau IP associé | `10.15.0.0/24`    | `10.25.0.0/28`   | `10.35.0.0/27`     | `10.45.0.0/24`    | `10.55.0.0/28`    | `10.65.0.0/28`    | `10.75.0.0/28`    | `10.85.0.0/24`    | `10.95.0.0/28`    |

---

- Quel client est dans quel VLAN

**- Site "Meow Origins" :**

| Machine - VLAN  | VLAN 10 `dev1`    | VLAN 20 `adm1`   | VLAN 30 `dir1`    | VLAN 40 `serv1`   | VLAN 50 `imp1`    | VLAN 60 `cam1`    | VLAN 70 `tv1`     | VLAN 80 `tel1`    | VLAN 90 `service1`|
| --------------  | ----------------- | ---------------- | ----------------- | ----------------- | ----------------- | ----------------- | ----------------- | ----------------- | ----------------- |
| `r1`            | ✅                | ✅               | ✅               | ✅                | ✅               | ✅               | ✅                | ✅               | ✅               | 
| `dev.lead1.O`   | ✅                | ❌               | ❌               | ❌                | ❌               | ❌               | ❌                | ❌               | ❌               |
| `dev.lead7.O`   | ✅                | ❌               | ❌               | ❌                | ❌               | ❌               | ❌                | ❌               | ❌               |
| `dev.1.O`       | ✅                | ❌               | ❌               | ❌                | ❌               | ❌               | ❌                | ❌               | ❌               |
| `dev.132.O`     | ✅                | ❌               | ❌               | ❌                | ❌               | ❌               | ❌                | ❌               | ❌               |
| `adm.res.O`     | ❌                | ✅               | ❌               | ❌                | ❌               | ❌               | ❌                | ❌               | ❌               |
| `adm.sys.O`     | ❌                | ✅               | ❌               | ❌                | ❌               | ❌               | ❌                | ❌               | ❌               |
| `adm.secu.O`    | ❌                | ✅               | ❌               | ❌                | ❌               | ❌               | ❌                | ❌               | ❌               |
| `pdg`           | ❌                | ❌               | ✅               | ❌                | ❌               | ❌               | ❌                | ❌               | ❌               |
| `sec.1.O`       | ❌                | ❌               | ✅               | ❌                | ❌               | ❌               | ❌                | ❌               | ❌               |
| `sec.2.O`       | ❌                | ❌               | ✅               | ❌                | ❌               | ❌               | ❌                | ❌               | ❌               |
| `sec.3.O`       | ❌                | ❌               | ✅               | ❌                | ❌               | ❌               | ❌                | ❌               | ❌               |
| `sec.4.O`       | ❌                | ❌               | ✅               | ❌                | ❌               | ❌               | ❌                | ❌               | ❌               |
| `sec.5.O`       | ❌                | ❌               | ✅               | ❌                | ❌               | ❌               | ❌                | ❌               | ❌               |
| `rh.1.O`        | ❌                | ❌               | ✅               | ❌                | ❌               | ❌               | ❌                | ❌               | ❌               |
| `rh.2.O`        | ❌                | ❌               | ✅               | ❌                | ❌               | ❌               | ❌                | ❌               | ❌               |
| `test.serv1.O`  | ❌                | ❌               | ❌               | ✅                | ❌               | ❌               | ❌                | ❌               | ❌               |
| `test.serv30.O` | ❌                | ❌               | ❌               | ✅                | ❌               | ❌               | ❌                | ❌               | ❌               |
| `test.db.O`     | ❌                | ❌               | ❌               | ✅                | ❌               | ❌               | ❌                | ❌               | ❌               |
| `prod.serv1.O`  | ❌                | ❌               | ❌               | ✅                | ❌               | ❌               | ❌                | ❌               | ❌               |
| `prod.serv2.O`  | ❌                | ❌               | ❌               | ✅                | ❌               | ❌               | ❌                | ❌               | ❌               |
| `prod.db.O`     | ❌                | ❌               | ❌               | ✅                | ❌               | ❌               | ❌                | ❌               | ❌               |
| `git.serv.O`    | ❌                | ❌               | ❌               | ✅                | ❌               | ❌               | ❌                | ❌               | ❌               |
| `imp.1.O`       | ❌                | ❌               | ❌               | ❌                | ✅               | ❌               | ❌                | ❌               | ❌               |
| `imp.2.O`       | ❌                | ❌               | ❌               | ❌                | ✅               | ❌               | ❌                | ❌               | ❌               |
| `imp.3.O`       | ❌                | ❌               | ❌               | ❌                | ✅               | ❌               | ❌                | ❌               | ❌               |
| `cam.1.O`       | ❌                | ❌               | ❌               | ❌                | ❌               | ✅               | ❌                | ❌               | ❌               |
| `cam.2.O`       | ❌                | ❌               | ❌               | ❌                | ❌               | ✅               | ❌                | ❌               | ❌               |
| `cam.3.O`       | ❌                | ❌               | ❌               | ❌                | ❌               | ✅               | ❌                | ❌               | ❌               |
| `cam.4.O`       | ❌                | ❌               | ❌               | ❌                | ❌               | ✅               | ❌                | ❌               | ❌               |
| `cam.5.O`       | ❌                | ❌               | ❌               | ❌                | ❌               | ✅               | ❌                | ❌               | ❌               |
| `tv.1.O`        | ❌                | ❌               | ❌               | ❌                | ❌               | ❌               | ✅                | ❌               | ❌               |
| `tv.2.O`        | ❌                | ❌               | ❌               | ❌                | ❌               | ❌               | ✅                | ❌               | ❌               |
| `tv.3.O`        | ❌                | ❌               | ❌               | ❌                | ❌               | ❌               | ✅                | ❌               | ❌               |
| `tv.4.O`        | ❌                | ❌               | ❌               | ❌                | ❌               | ❌               | ✅                | ❌               | ❌               |
| `tel.1.O`       | ❌                | ❌               | ❌               | ❌                | ❌               | ❌               | ❌                | ✅               | ❌               |
| `tel.2.O`       | ❌                | ❌               | ❌               | ❌                | ❌               | ❌               | ❌                | ✅               | ❌               |
| `tel.3.O`       | ❌                | ❌               | ❌               | ❌                | ❌               | ❌               | ❌                | ✅               | ❌               |
| `tel.4.O`       | ❌                | ❌               | ❌               | ❌                | ❌               | ❌               | ❌                | ✅               | ❌               |
| `tel.5.O`       | ❌                | ❌               | ❌               | ❌                | ❌               | ❌               | ❌                | ✅               | ❌               |
| `dns.dev.meow`  | ❌                | ❌               | ❌               | ❌                | ❌               | ❌               | ❌                | ❌               | ✅               |
| `dhcp.tp8.O`    | ❌                | ❌               | ❌               | ❌                | ❌               | ❌               | ❌                | ❌               | ✅               |


**- Site "Meow and Beyond" :**

| Machine - VLAN        | VLAN 15 `dev2`    | VLAN 25 `adm2`   | VLAN 35 `dir2`    | VLAN 45 `serv2`   | VLAN 55 `imp2`    | VLAN 65 `cam2`    | VLAN 75 `tv2`     | VLAN 85 `tel2`    | VLAN 95 `service2`|
| --------------------- | ----------------- | ---------------- | ----------------- | ----------------- | ----------------- | ----------------- | ----------------- | ----------------- | ----------------- |
| `r2`                  | ✅                | ✅              | ✅               | ✅                | ✅                | ✅               | ✅                | ✅               | ✅               |
| `devlead1.bey.meo`    | ✅                | ❌              | ❌               | ❌                | ❌                | ❌               | ❌                | ❌               | ❌               |
| `dev1.bey.meo`        | ✅                | ❌              | ❌               | ❌                | ❌                | ❌               | ❌                | ❌               | ❌               |
| `admres.bey.meo`      | ❌                | ✅              | ❌               | ❌                | ❌                | ❌               | ❌                | ❌               | ❌               |
| `admsys.bey.meo`      | ❌                | ✅              | ❌               | ❌                | ❌                | ❌               | ❌                | ❌               | ❌               |
| `sec1.bey.meo`        | ❌                | ❌              | ✅               | ❌                | ❌                | ❌               | ❌                | ❌               | ❌               |
| `sec2.bey.meo`        | ❌                | ❌              | ✅               | ❌                | ❌                | ❌               | ❌                | ❌               | ❌               |
| `testserv1.bey.meo`   | ❌                | ❌              | ❌               | ✅                | ❌                | ❌               | ❌                | ❌               | ❌               |
| `testserv30.bey.meo`  | ❌                | ❌              | ❌               | ✅                | ❌                | ❌               | ❌                | ❌               | ❌               |
| `testdb.bey.meo`      | ❌                | ❌              | ❌               | ✅                | ❌                | ❌               | ❌                | ❌               | ❌               |
| `imp1.bey.meo`        | ❌                | ❌              | ❌               | ❌                | ✅                | ❌               | ❌                | ❌               | ❌               |
| `imp2.bey.meo`        | ❌                | ❌              | ❌               | ❌                | ✅                | ❌               | ❌                | ❌               | ❌               |
| `cam1.bey.meo`        | ❌                | ❌              | ❌               | ❌                | ❌                | ✅               | ❌                | ❌               | ❌               |
| `cam2.bey.meo`        | ❌                | ❌              | ❌               | ❌                | ❌                | ✅               | ❌                | ❌               | ❌               |
| `tv1.bey.meo`         | ❌                | ❌              | ❌               | ❌                | ❌                | ❌               | ✅                | ❌               | ❌               |
| `tv2.bey.meo`         | ❌                | ❌              | ❌               | ❌                | ❌                | ❌               | ✅                | ❌               | ❌               |
| `tv3.bey.meo`         | ❌                | ❌              | ❌               | ❌                | ❌                | ❌               | ✅                | ❌               | ❌               |
| `tv4.bey.meo`         | ❌                | ❌              | ❌               | ❌                | ❌                | ❌               | ✅                | ❌               | ❌               |
| `tel1.bey.meo`        | ❌                | ❌              | ❌               | ❌                | ❌                | ❌               | ❌                | ✅               | ❌               |
| `tel2.bey.meo`        | ❌                | ❌              | ❌               | ❌                | ❌                | ❌               | ❌                | ✅               | ❌               |
| `dhcp.bey.meo`        | ❌                | ❌              | ❌               | ❌                | ❌                | ❌               | ❌                | ❌               | ✅               |



🌞 **Config de toutes les machines**

- un `show-run` pour les équipements réseau
  - routeurs et switches

```bash
R1#sh running-config
Building configuration...

interface FastEthernet0/0
 ip address dhcp
 ip nat outside
 ip virtual-reassembly
 duplex auto
 speed auto
!
interface FastEthernet0/1
 ip address 10.8.1.1 255.255.255.252
 duplex auto
 speed auto
!
interface FastEthernet1/0
 no ip address
 ip helper-address 10.90.0.2
 ip nat inside
 ip virtual-reassembly
 duplex auto
 speed auto
!
interface FastEthernet1/0.10
 encapsulation dot1Q 10
 ip address 10.10.1.254 255.255.254.0
 ip helper-address 10.90.0.2
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet1/0.20
 encapsulation dot1Q 20
 ip address 10.20.0.14 255.255.255.240
 ip helper-address 10.90.0.2
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet1/0.30
 encapsulation dot1Q 30
 ip address 10.30.0.30 255.255.255.224
 ip helper-address 10.90.0.2
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet1/0.40
 encapsulation dot1Q 40
 ip address 10.40.0.254 255.255.255.0
 ip helper-address 10.90.0.2
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet1/0.50
 encapsulation dot1Q 50
 ip address 10.50.0.14 255.255.255.240
 ip helper-address 10.90.0.2
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet1/0.60
 encapsulation dot1Q 60
 ip address 10.60.0.14 255.255.255.240
 ip helper-address 10.90.0.2
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet1/0.70
 encapsulation dot1Q 70
 ip address 10.70.0.14 255.255.255.240
 ip helper-address 10.90.0.2
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet1/0.80
 encapsulation dot1Q 80
 ip address 10.80.1.254 255.255.254.0
 ip helper-address 10.90.0.2
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet1/0.90
 encapsulation dot1Q 90
 ip address 10.90.0.14 255.255.255.240
 ip helper-address 10.90.0.2
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet2/0
 no ip address
 shutdown
 duplex auto
 speed auto
!
ip forward-protocol nd
ip route 10.15.0.0 255.255.255.0 10.8.1.2
ip route 10.25.0.0 255.255.255.240 10.8.1.2
ip route 10.35.0.0 255.255.255.224 10.8.1.2
ip route 10.45.0.0 255.255.255.0 10.8.1.2
ip route 10.55.0.0 255.255.255.240 10.8.1.2
ip route 10.65.0.0 255.255.255.240 10.8.1.2
ip route 10.75.0.0 255.255.255.240 10.8.1.2
ip route 10.85.0.0 255.255.255.0 10.8.1.2
ip route 10.95.0.0 255.255.255.240 10.8.1.2
!
!
no ip http server
no ip http secure-server
ip nat inside source list 1 interface FastEthernet0/0 overload
!
access-list 1 permit any
```

```bash

R2#sh running-config
Building configuration...

interface FastEthernet0/0
 ip address dhcp
 ip nat outside
 ip virtual-reassembly
 duplex auto
 speed auto
!
interface FastEthernet0/1
 ip address 10.8.1.2 255.255.255.252
 duplex auto
 speed auto
!
interface FastEthernet1/0
 no ip address
 ip helper-address 10.95.0.1
 ip nat inside
 ip virtual-reassembly
 duplex auto
 speed auto
!
interface FastEthernet1/0.15
 encapsulation dot1Q 15
 ip address 10.15.0.254 255.255.255.0
 ip helper-address 10.95.0.1
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet1/0.25
 encapsulation dot1Q 25
 ip address 10.25.0.14 255.255.255.240
 ip helper-address 10.95.0.1
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet1/0.35
 encapsulation dot1Q 35
 ip address 10.35.0.30 255.255.255.224
 ip helper-address 10.95.0.1
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet1/0.45
 encapsulation dot1Q 45
 ip address 10.45.0.254 255.255.255.0
 ip helper-address 10.95.0.1
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet1/0.55
 encapsulation dot1Q 55
 ip address 10.55.0.14 255.255.255.240
 ip helper-address 10.95.0.1
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet1/0.65
 encapsulation dot1Q 65
 ip address 10.65.0.14 255.255.255.240
 ip helper-address 10.95.0.1
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet1/0.75
 encapsulation dot1Q 75
 ip address 10.75.0.14 255.255.255.240
 ip helper-address 10.95.0.1
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet1/0.85
 encapsulation dot1Q 85
 ip address 10.85.0.254 255.255.255.0
 ip helper-address 10.95.0.1
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet1/0.95
 encapsulation dot1Q 95
 ip address 10.95.0.14 255.255.255.240
 ip helper-address 10.95.0.1
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet2/0
 no ip address
 shutdown
 duplex auto
 speed auto
!
ip forward-protocol nd
ip route 0.0.0.0 0.0.0.0 FastEthernet0/1
ip route 10.10.0.0 255.255.240.0 10.8.1.1
ip route 10.10.0.0 255.255.254.0 10.8.1.1
ip route 10.20.0.0 255.255.255.240 10.8.1.1
ip route 10.30.0.0 255.255.255.224 10.8.1.1
ip route 10.40.0.0 255.255.255.0 10.8.1.1
ip route 10.50.0.0 255.255.255.240 10.8.1.1
ip route 10.60.0.0 255.255.255.240 10.8.1.1
ip route 10.70.0.0 255.255.255.240 10.8.1.1
ip route 10.80.0.0 255.255.254.0 10.8.1.1
ip route 10.90.0.0 255.255.255.240 10.8.1.1
!
!
no ip http server
no ip http secure-server
ip nat inside source list 1 interface FastEthernet0/0 overload
!
access-list 1 permit any
```

```bash
IOU-O#sh running-config
Building configuration...

interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/2
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/3
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet1/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet1/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
```
```bash
Plateforme-O#sh running-config
Building configuration...

interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/1
 switchport access vlan 40
 switchport mode access
!
interface Ethernet0/2
 switchport access vlan 40
 switchport mode access
!
interface Ethernet0/3
 switchport access vlan 40
 switchport mode access
!
interface Ethernet1/0
 switchport access vlan 40
 switchport mode access
!
interface Ethernet1/1
 switchport access vlan 40
 switchport mode access
!
interface Ethernet1/2
 switchport access vlan 40
 switchport mode access
!
interface Ethernet1/3
 switchport access vlan 40
 switchport mode access
!
interface Ethernet2/0
 switchport access vlan 40
 switchport mode access
!
interface Ethernet2/1
 switchport access vlan 40
 switchport mode access
!
interface Ethernet2/2
 switchport access vlan 40
 switchport mode access
!
interface Ethernet2/3
 switchport access vlan 40
 switchport mode access
!
```
```bash
service-O#sh running-config
Building configuration...

interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/1
 switchport access vlan 90
 switchport mode access
!
interface Ethernet0/2
 switchport access vlan 90
 switchport mode access
!
```

```bash

RDC#sh running-config
Building configuration...

interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/1
 switchport access vlan 30
 switchport mode access
!
interface Ethernet0/2
 switchport access vlan 30
 switchport mode access
!
interface Ethernet0/3
 switchport access vlan 30
 switchport mode access
!
interface Ethernet1/0
 switchport access vlan 50
 switchport mode access
!
interface Ethernet1/1
 switchport access vlan 60
 switchport mode access
!
interface Ethernet1/2
 switchport access vlan 80
 switchport mode access
!
interface Ethernet1/3
 switchport access vlan 70
 switchport mode access
!
interface Ethernet2/0
 switchport access vlan 70
 switchport mode access
!
```
```bash
E1#sh running-config
Building configuration...

interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/1
 switchport access vlan 30
 switchport mode access
!
interface Ethernet0/2
 switchport access vlan 30
 switchport mode access
!
interface Ethernet0/3
 switchport access vlan 50
 switchport mode access
!
interface Ethernet1/0
 switchport access vlan 60
 switchport mode access
!
interface Ethernet1/1
 switchport access vlan 60
 switchport mode access
!
interface Ethernet1/2
 switchport access vlan 70
 switchport mode access
!
interface Ethernet1/3
 switchport access vlan 80
 switchport mode access
!
interface Ethernet2/0
 switchport access vlan 80
 switchport mode access
!
interface Ethernet2/1
 switchport access vlan 10
 switchport mode access
!
interface Ethernet2/2
 switchport access vlan 10
 switchport mode access
!
```
```bash
E2#sh running-config
Building configuration...

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
 switchport access vlan 30
 switchport mode access
!
interface Ethernet1/0
 switchport access vlan 50
 switchport mode access
!
interface Ethernet1/1
 switchport access vlan 60
 switchport mode access
!
interface Ethernet1/2
 switchport access vlan 60
 switchport mode access
!
interface Ethernet1/3
 switchport access vlan 70
 switchport mode access
!
interface Ethernet2/0
 switchport access vlan 80
 switchport mode access
!
interface Ethernet2/1
 switchport access vlan 30
 switchport mode access
!
interface Ethernet2/2
 switchport access vlan 80
 switchport mode access
!
interface Ethernet2/3
 switchport access vlan 30
 switchport mode access
!
interface Ethernet3/0
 switchport access vlan 80
 switchport mode access
!
interface Ethernet3/1
 switchport access vlan 20
 switchport mode access
!
interface Ethernet3/2
 switchport access vlan 20
 switchport mode access
!
interface Ethernet3/3
 switchport access vlan 20
 switchport mode access
!
```
```bash
R2#sh running-config
Building configuration...

interface FastEthernet0/0
 ip address dhcp
 ip nat outside
 ip virtual-reassembly
 duplex auto
 speed auto
!
interface FastEthernet0/1
 ip address 10.8.1.2 255.255.255.252
 duplex auto
 speed auto
!
interface FastEthernet1/0
 no ip address
 ip helper-address 10.95.0.1
 ip nat inside
 ip virtual-reassembly
 duplex auto
 speed auto
!
interface FastEthernet1/0.15
 encapsulation dot1Q 15
 ip address 10.15.0.254 255.255.255.0
 ip helper-address 10.95.0.1
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet1/0.25
 encapsulation dot1Q 25
 ip address 10.25.0.14 255.255.255.240
 ip helper-address 10.95.0.1
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet1/0.35
 encapsulation dot1Q 35
 ip address 10.35.0.30 255.255.255.224
 ip helper-address 10.95.0.1
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet1/0.45
 encapsulation dot1Q 45
 ip address 10.45.0.254 255.255.255.0
 ip helper-address 10.95.0.1
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet1/0.55
 encapsulation dot1Q 55
 ip address 10.55.0.14 255.255.255.240
 ip helper-address 10.95.0.1
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet1/0.65
 encapsulation dot1Q 65
 ip address 10.65.0.14 255.255.255.240
 ip helper-address 10.95.0.1
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet1/0.75
 encapsulation dot1Q 75
 ip address 10.75.0.14 255.255.255.240
 ip helper-address 10.95.0.1
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet1/0.85
 encapsulation dot1Q 85
 ip address 10.85.0.254 255.255.255.0
 ip helper-address 10.95.0.1
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet1/0.95
 encapsulation dot1Q 95
 ip address 10.95.0.14 255.255.255.240
 ip helper-address 10.95.0.1
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet2/0
 no ip address
 shutdown
 duplex auto
 speed auto
!
ip forward-protocol nd
ip route 0.0.0.0 0.0.0.0 FastEthernet0/1
ip route 10.10.0.0 255.255.240.0 10.8.1.1
ip route 10.10.0.0 255.255.254.0 10.8.1.1
ip route 10.20.0.0 255.255.255.240 10.8.1.1
ip route 10.30.0.0 255.255.255.224 10.8.1.1
ip route 10.40.0.0 255.255.255.0 10.8.1.1
ip route 10.50.0.0 255.255.255.240 10.8.1.1
ip route 10.60.0.0 255.255.255.240 10.8.1.1
ip route 10.70.0.0 255.255.255.240 10.8.1.1
ip route 10.80.0.0 255.255.254.0 10.8.1.1
ip route 10.90.0.0 255.255.255.240 10.8.1.1
!
!
no ip http server
no ip http secure-server
ip nat inside source list 1 interface FastEthernet0/0 overload
!
access-list 1 permit any
```
```bash
IOU-B#sh running-config
Building configuration...


interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/2
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/3
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet1/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
```
```bash
Plateforme-B#sh running-config
Building configuration...

interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/1
 switchport access vlan 45
 switchport mode access
!
interface Ethernet0/2
 switchport access vlan 45
 switchport mode access
!
interface Ethernet0/3
 switchport access vlan 45
 switchport mode access
!
```
```bash
service-B#sh running-config
Building configuration...

interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/1
 switchport access vlan 95
 switchport mode access
!
```
```bash
bat1#sh running-config
Building configuration...

interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/1
 switchport access vlan 35
 switchport mode access
!
interface Ethernet0/2
 switchport access vlan 35
 switchport mode access
!
interface Ethernet0/3
 switchport access vlan 25
 switchport mode access
!
interface Ethernet1/0
 switchport access vlan 25
 switchport mode access
!
interface Ethernet1/1
 switchport access vlan 55
 switchport mode access
!
interface Ethernet1/2
 switchport access vlan 65
 switchport mode access
!
interface Ethernet1/3
 switchport access vlan 75
 switchport mode access
!
interface Ethernet2/0
 switchport access vlan 75
 switchport mode access
!
interface Ethernet2/1
 switchport access vlan 85
 switchport mode access
!
```
```bash
bat2#sh running-config
Building configuration...

interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/1
 switchport access vlan 15
 switchport mode access
!
interface Ethernet0/2
 switchport access vlan 15
 switchport mode access
!
interface Ethernet0/3
 switchport access vlan 55
 switchport mode access
!
interface Ethernet1/0
 switchport access vlan 65
 switchport mode access
!
interface Ethernet1/1
 switchport access vlan 75
 switchport mode access
!
interface Ethernet1/2
 switchport access vlan 75
 switchport mode access
!
interface Ethernet1/3
 switchport access vlan 85
 switchport mode access
!
```

- la suite des étapes pour les machines Linux
  - vous ne configurez QUE le serveur DHCP et DNS pour la partie Linux

### dhcp.ori.meo
```bash
[manon@dhcp ~]$ sudo dnf install dhcp-server -y
```
```bash
[manon@dhcp ~]$ sudo cat /etc/dhcp/dhcpd.conf
[sudo] password for manon:
# DHCP Server Configuration file.
# create new
# specify domain name
option domain-name     "srv.world";
# specify DNS server's hostname or IP address
option domain-name-servers     10.90.0.1;
# default lease time
default-lease-time 600;
# max lease time
max-lease-time 7200;
# this DHCP server to be declared valid
authoritative;
# specify network address and subnetmask
subnet 10.10.0.0 netmask 255.255.254.0 {
    range dynamic-bootp 10.10.0.1 10.10.1.253;
    option broadcast-address 10.10.1.255;
    option routers 10.10.1.254;
}

subnet 10.20.0.0 netmask 255.255.255.240 {
    range dynamic-bootp 10.20.0.1 10.20.0.13;
    option broadcast-address 10.20.0.15;
    option routers 10.20.0.14;
}

subnet 10.30.0.0 netmask 255.255.224.0 {
    range dynamic-bootp 10.30.0.1 10.30.0.29;
    option broadcast-address 10.30.0.31;
    option routers 10.30.0.30;
}

subnet 10.40.0.0 netmask 255.255.255.0 {
    range dynamic-bootp 10.40.0.1 10.40.0.253;
    option broadcast-address 10.40.0.255;
    option routers 10.40.0.254;
}

subnet 10.50.0.0 netmask 255.255.255.240 {
    range dynamic-bootp 10.50.0.1 10.50.0.13;
    option broadcast-address 10.50.0.15;
    option routers 10.50.0.14;
}

subnet 10.60.0.0 netmask 255.255.255.240 {
    range dynamic-bootp 10.60.0.1 10.60.0.13;
    option broadcast-address 10.60.0.15;
    option routers 10.60.0.14;
}

subnet 10.70.0.0 netmask 255.255.255.240 {
    range dynamic-bootp 10.70.0.1 10.70.0.13;
    option broadcast-address 10.70.0.15;
    option routers 10.70.0.14;
}

subnet 10.80.0.0 netmask 255.255.254.0 {
    range dynamic-bootp 10.80.0.1 10.80.1.253;
    option broadcast-address 10.80.1.255;
    option routers 10.80.1.254;
}

subnet 10.90.0.0 netmask 255.255.255.240 {
    range dynamic-bootp 10.90.0.1 10.90.0.13;
    option broadcast-address 10.90.0.15;
    option routers 10.90.0.14;
}
```
```bash
[manon@dhcp ~]$ sudo systemctl enable --now dhcpd
[manon@dhcp ~]$ sudo firewall-cmd --add-service=dhcp
success
[manon@dhcp ~]$ sudo firewall-cmd --runtime-to-permanent
success
[manon@dhcp ~]$ systemctl status dhcpd
● dhcpd.service - DHCPv4 Server Daemon
     Loaded: loaded (/usr/lib/systemd/system/dhcpd.service; enabled; preset: disabled)
     Active: active (running) since Tue 2023-12-19 18:47:56 CET; 2h 55min ago
       Docs: man:dhcpd(8)
             man:dhcpd.conf(5)
   Main PID: 803 (dhcpd)
     Status: "Dispatching packets..."
      Tasks: 1 (limit: 11038)
     Memory: 7.6M
        CPU: 10ms
     CGroup: /system.slice/dhcpd.service
             └─803 /usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd.conf -user dhcpd -group dhcpd --no-pid

Dec 19 18:47:56 dhcp.ori.meo dhcpd[803]:
Dec 19 18:47:56 dhcp.ori.meo dhcpd[803]: No subnet declaration for enp0s3 (no IPv4 addresses).
Dec 19 18:47:56 dhcp.ori.meo dhcpd[803]: ** Ignoring requests on enp0s3.  If this is not what
Dec 19 18:47:56 dhcp.ori.meo dhcpd[803]:    you want, please write a subnet declaration
Dec 19 18:47:56 dhcp.ori.meo dhcpd[803]:    in your dhcpd.conf file for the network segment
Dec 19 18:47:56 dhcp.ori.meo dhcpd[803]:    to which interface enp0s3 is attached. **
Dec 19 18:47:56 dhcp.ori.meo dhcpd[803]:
Dec 19 18:47:56 dhcp.ori.meo dhcpd[803]: Sending on   Socket/fallback/fallback-net
Dec 19 18:47:56 dhcp.ori.meo dhcpd[803]: Server starting service.
Dec 19 18:47:56 dhcp.ori.meo systemd[1]: Started DHCPv4 Server Daemon.
```

### dhcp.bey.meo
```bash
[manon@dhcp ~]$ sudo systemctl enable --now dhcpd
[manon@dhcp ~]$ sudo firewall-cmd --add-service=dhcp
success
[manon@dhcp ~]$ sudo firewall-cmd --runtime-to-permanent
success
[manon@dhcp ~]$ systemctl status dhcpd
● dhcpd.service - DHCPv4 Server Daemon
     Loaded: loaded (/usr/lib/systemd/system/dhcpd.service; enabled; preset: disabled)
     Active: active (running) since Tue 2023-12-19 19:06:56 CET; 2h 39min ago
       Docs: man:dhcpd(8)
             man:dhcpd.conf(5)
   Main PID: 801 (dhcpd)
     Status: "Dispatching packets..."
      Tasks: 1 (limit: 11038)
     Memory: 7.4M
        CPU: 20ms
     CGroup: /system.slice/dhcpd.service
             └─801 /usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd.conf -user dhcpd -group dhcpd --no-pid

Dec 19 21:23:40 dhcp.bey.meo dhcpd[801]: DHCPREQUEST for 10.85.0.1 (10.95.0.1) from 00:50:79:66:68:34 (tel1.bey.meo) via 10.85.0.254
Dec 19 21:23:40 dhcp.bey.meo dhcpd[801]: DHCPACK on 10.85.0.1 to 00:50:79:66:68:34 (tel1.bey.meo) via 10.85.0.254
Dec 19 21:28:42 dhcp.bey.meo dhcpd[801]: DHCPREQUEST for 10.85.0.1 (10.95.0.1) from 00:50:79:66:68:34 (tel1.bey.meo) via 10.85.0.254
Dec 19 21:28:42 dhcp.bey.meo dhcpd[801]: DHCPACK on 10.85.0.1 to 00:50:79:66:68:34 (tel1.bey.meo) via 10.85.0.254
Dec 19 21:33:43 dhcp.bey.meo dhcpd[801]: DHCPREQUEST for 10.85.0.1 (10.95.0.1) from 00:50:79:66:68:34 (tel1.bey.meo) via 10.85.0.254
Dec 19 21:33:43 dhcp.bey.meo dhcpd[801]: DHCPACK on 10.85.0.1 to 00:50:79:66:68:34 (tel1.bey.meo) via 10.85.0.254
Dec 19 21:38:44 dhcp.bey.meo dhcpd[801]: DHCPREQUEST for 10.85.0.1 (10.95.0.1) from 00:50:79:66:68:34 (tel1.bey.meo) via 10.85.0.254
Dec 19 21:38:44 dhcp.bey.meo dhcpd[801]: DHCPACK on 10.85.0.1 to 00:50:79:66:68:34 (tel1.bey.meo) via 10.85.0.254
Dec 19 21:43:46 dhcp.bey.meo dhcpd[801]: DHCPREQUEST for 10.85.0.1 (10.95.0.1) from 00:50:79:66:68:34 (tel1.bey.meo) via 10.85.0.254
Dec 19 21:43:46 dhcp.bey.meo dhcpd[801]: DHCPACK on 10.85.0.1 to 00:50:79:66:68:34 (tel1.bey.meo) via 10.85.0.254
```

### dns.ori.meo
```bash
[manon@dhcp ~]$ sudo dnf install -y bind bind-utils
```
```bash
[manon@dns etc]$ sudo cat /etc/named.conf
[sudo] password for manon:
options {
        listen-on port 53 {any; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        secroots-file   "/var/named/data/named.secroots";
        recursing-file  "/var/named/data/named.recursing";
        allow-query     { any; };

        /*
         - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
         - If you are building a RECURSIVE (caching) DNS server, you need to enable
           recursion.
         - If your recursive DNS server has a public IP address, you MUST enable access
           control to limit queries to your legitimate users. Failing to do so will
           cause your server to become part of large scale DNS amplification
           attacks. Implementing BCP38 within your network would greatly
           reduce such attack surface
        */
        recursion yes;

        allow-new-zones yes;
        dnssec-validation yes;

        managed-keys-directory "/var/named/dynamic";
        geoip-directory "/usr/share/GeoIP";

        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";

        /* https://fedoraproject.org/wiki/Changes/CryptoPolicy */
        include "/etc/crypto-policies/back-ends/bind.config";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};
zone "." IN {
        type hint;
        file "named.ca";
};
zone "ori.meo" IN {
        type master;
        file "ori.meo.db";
        allow-update {none;};
        allow-query {any;};
};
zone "bey.meo" IN {
        type master;
        file "bey.meo.db";
        allow-update {none;};
        allow-query {any;};
};
include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```
```bash
[manon@dns etc]$ sudo cat /var/named/ori.meo.db
$TTL 86400
@ IN SOA dns.ori.meo. admin.ori.meo. (
    2019061800 ;Serial
    3600 ;Refresh
    1800 ;Retry
    604800 ;Expire
    86400 ;Minimum TTL
)

; Infos sur le serveur DNS lui-même (NS = NameServer)
@ IN NS dns.ori.meo.

; Enregistrements DNS pour faire correspondre des noms à des IPs
dns           IN A 10.90.0.1
testserv1     IN A 10.40.0.1
testserv30    IN A 10.40.0.2
testdb        IN A 10.40.0.3
prodserv1     IN A 10.40.0.4
prodserv2     IN A 10.40.0.5
proddb        IN A 10.40.0.6
gitserv       IN A 10.40.0.7
dhcp          IN A 10.90.0.2
```
```bash
[manon@dns etc]$ sudo cat /var/named/bey.meo.db
$TTL 86400
@ IN SOA dns.ori.meo. admin.ori.meo. (
    2019061800 ;Serial
    3600 ;Refresh
    1800 ;Retry
    604800 ;Expire
    86400 ;Minimum TTL
)

; Infos sur le serveur DNS lui même (NS = NameServer)
@ IN NS dns.ori.meo.

; Enregistrements DNS pour faire correspondre des noms à des IPs
dns           IN A 10.95.0.1
testserv1     IN A 10.45.0.1
testserv30    IN A 10.45.0.2
testdb        IN A 10.45.0.3
dhcp          IN A 10.95.0.1
```
```bash
[manon@dns etc]$ sudo firewall-cmd --permanent --add-service=dns
[manon@dns etc]$ sudo firewall-cmd --permanent --add-port=53/tcp
success
[manon@dns etc]$ sudo firewall-cmd --permanent --add-port=53/udp
success
[manon@dns etc]$ sudo firewall-cmd --reload
successs
```

### Test
```bash
proddb.ori.meo> ip dhcp
DDORA IP 10.40.0.6/24 GW 10.40.0.254

proddb.ori.meo> sh ip

NAME        : proddb.ori.meo
IP/MASK     : 10.40.0.6/24
GATEWAY     : 10.40.0.254
DNS         : 10.90.0.1
DHCP SERVER : 10.90.0.2
DHCP LEASE  : 500, 600/300/525
DOMAIN NAME : srv.world
MAC         : 00:50:79:66:68:06
LPORT       : 20174
RHOST:PORT  : 127.0.0.1:20175
MTU         : 1500

```
```bash
pdg.ori.meo> ip dhcp
DDORA IP 10.30.0.2/19 GW 10.30.0.30

pdg.ori.meo> sh ip

NAME        : pdg.ori.meo
IP/MASK     : 10.30.0.1/19
GATEWAY     : 10.30.0.30
DNS         : 10.90.0.1
DHCP SERVER : 10.90.0.2
DHCP LEASE  : 590, 600/300/525
DOMAIN NAME : srv.world
MAC         : 00:50:79:66:68:17
LPORT       : 20172
RHOST:PORT  : 127.0.0.1:20173
MTU         : 1500
```
```bash
pdg.ori.meo> ping gitserv.ori.meo
gitserv.ori.meo resolved to 10.40.0.7

84 bytes from 10.40.0.7 icmp_seq=1 ttl=63 time=29.904 ms
84 bytes from 10.40.0.7 icmp_seq=2 ttl=63 time=21.262 ms
84 bytes from 10.40.0.7 icmp_seq=3 ttl=63 time=18.649 ms
```
```bash
pdg.ori.meo> ping ynov.com
ynov.com resolved to 172.67.74.226
```
```bash
tel1.bey.meo> ip dhcp
DORA IP 10.85.0.1/24 GW 10.85.0.254

tel1.bey.meo> sh ip

NAME        : tel1.bey.meo[1]
IP/MASK     : 10.85.0.1/24
GATEWAY     : 10.85.0.254
DNS         : 10.90.0.1
DHCP SERVER : 10.95.0.1
DHCP LEASE  : 463, 491/245/429
DOMAIN NAME : srv.world
MAC         : 00:50:79:66:68:34
LPORT       : 20164
RHOST:PORT  : 127.0.0.1:20165
MTU         : 1500
```
```bash
tel1.bey.meo> ping gitserv.ori.meo
gitserv.ori.meo resolved to 10.40.0.7

84 bytes from 10.40.0.7 icmp_seq=1 ttl=62 time=40.837 ms
84 bytes from 10.40.0.7 icmp_seq=2 ttl=62 time=37.032 ms
```
```bash
tel1.bey.meo> ping youtube.com
youtube.com resolved to 142.250.178.142
```