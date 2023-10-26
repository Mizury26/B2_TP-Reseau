# TP3 DEV : Python et réseau

Dans ce premier TP, on va se familiariser avec quelques notions Python, en faisant des trucs de réseau. J'VOUS JURE J'ESSAIE DE PAS ÊTRE CHIANT.

Pas de VM quedal, on fait simple pour le premier TP.

Créez un dépôt git dédié à ce TP. Dans votre dépôt principal, un dossier `tp3`, dedans un fichier `README.md` qui contient un lien vers le dépôt dédié à ce TP.

- [TP3 DEV : Python et réseau](#tp3-dev--python-et-réseau)
- [I. Ping](#i-ping)
- [II. DNS](#ii-dns)
- [III. Get your IP](#iii-get-your-ip)
- [IV. Mix](#iv-mix)
- [V. Deploy](#v-deploy)

# I. Ping

Y'a un truc qu'on aime pas du tout, dans n'importe quel langage, c'est mélanger les langages. Genre depuis un code Python, t'appelles du code PHP. Berk. Dégueu.

Un truc super mal vu donc, c'est d'utilise la lib `os` et la fonction `system()` qu'elle contient pour exécuter des commandes shell depuis Python.

C'est exactement ce qu'on va faire ici.

> Sans déc, on évite vraiment, c'est nul. Mais là on va faire des `ping` et pour faire ça en Python simplement, bah ça reste le mieux.

🌞 **`ping_simple.py`**

- importez la fonction `system` de la lib `os`
- utilisez-la pour effectuer un `ping 8.8.8.8`

---

Il est possible de récupérer dans le codes arguments passés quand on exécute le code. On utilise la liste `argv` de la lib native `sys`.

Par exemple, avec un fichier `test.py` :

```python
from sys import argv

print(f"Premier argument : {argv[0]}. Deuxième argument : {argv[1]}.")
```

On peut alors faire :

```bash
$ python test.py meooo b2
Premier argument : meooo. Deuxième argument : b2.
```

🌞 **`ping_arg.py`**

- importez la liste `argv` de la lib `sys`
  - cette liste contient les arguments passés au script à l'exécution
- effectuez un `ping` vers l'IP fournie en argument quand on exécute le code

```bash
# on pourra donc faire
$ python ping_arg.py 8.8.8.8
[...]
```

🌞 **`is_up.py`**

- même code que `ping_arg.py` mais il doit
  - soit juste afficher "UP !" si la machine répond au ping
  - soit juste afficher "DOWN !" si la machine ne répond pas

```bash
# on pourra donc faire
$ python is_up.py 8.8.8.8
UP !
```

# II. DNS

Hého c'est le premier TP. On fait des ptits `if` et des ptites fonctions. J'trouve ce que je peux pour coller au réseau en trame de fond.

🌞 **`lookup.py`**

- utilisez la fonction `gethostbyname()` de la lib `socket`
- permet de résoudre un nom de domaine passé en argument à la fonction

```bash
# vous devez permettre
$ python lookup.py ynov.com
104.26.11.233
```

# III. Get your IP

Ici on va utiliser une lib externe, histoire de passer une petite commande `pip`.

Installez la librairie `psutil` avec la commande `pip install psutil`.

🌞 **`get_ip.py`**

- utilisez la fonction `net_if_addrs()` de la librairie `psutil`
- votre code doit afficher l'adresse IP de votre carte WiFi

```bash
$ python get_ip.py
192.168.1.29
```

# IV. Mix

Vous allez réutiliser le contenu de `lookup.py`, `is_up.py` et `get_ip.py`. Le but, une seule feuille de code pour les gouverner toutes :

🌞 **`network.py`**

- doit contenir une fonction `lookup()` qui prend en argument un nom de domaine et retourne l'IP qui correspond
- doit contenir une fonction `ping()` qui prend en argument une IP et retourne "UP !" ou "DOWN !"
- doit contenir une fonction `ip()` qui prend rien en argument et retourne l'IP de la carte WiFi
- **ne doit contenir qu'un seul `print()`**

```bash
$ python network.py lookup ynov.com
104.26.11.233

$ python network.py ping 8.8.8.8
UP !

$ python network.py ip
192.168.1.29

$ python network.py ranlarjan
'ranlarjan' is not an available command. Déso.
```

# V. Deploy

Déployez-moi ça dans une VM Rocky :

- git clone du dépôt git de code, dans la VM
- install/mise à jour de Python si nécessaire
- exécution du code, le code doit fonctionner normalement