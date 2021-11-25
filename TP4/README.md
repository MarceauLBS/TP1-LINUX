# TP4 : Une distribution orientée serveur

# Sommaire

- [TP4 : Une distribution orientée serveur](#tp4--une-distribution-orientée-serveur)
- [Sommaire](#sommaire)
- [I. Install de Rocky Linux](#i-install-de-rocky-linux)
- [II. Checklist](#ii-checklist)
- [III. Mettre en place un service](#iii-mettre-en-place-un-service)
  - [1. Intro NGINX](#1-intro-nginx)
  - [2. Install](#2-install)
  - [3. Analyse](#3-analyse)
  - [4. Visite du service web](#4-visite-du-service-web)
  - [5. Modif de la conf du serveur web](#5-modif-de-la-conf-du-serveur-web)

# I. Install de Rocky Linux

# II. Checklist

🌞 **Choisissez et définissez une IP à la VM**

- vous allez devoir configurer l'interface host-only
- ce sera nécessaire pour pouvoir SSH dans la VM et donc écrire le compte-rendu
- je veux, dans le compte-rendu, le contenu de votre fichier de conf, et le résultat d'un `ip a` pour me prouver que les changements on pris effet

---
```
[marceau@localhost ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:18:70:ce brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
       valid_lft 86306sec preferred_lft 86306sec
    inet6 fe80::a00:27ff:fe18:70ce/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:5c:75:60 brd ff:ff:ff:ff:ff:ff
    inet 10.200.1.10/24 brd 10.200.1.255 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe5c:7560/64 scope link
       valid_lft forever preferred_lft forever
```

➜ **Connexion SSH fonctionnelle**

Vous pouvez vous connecter en SSH à la VM. Cela implique :

- la VM a une carte réseau avec une IP locale
- votre PC peut joindre cette IP
- la VM a un serveur SSH qui est accessible derrière l'un des ports réseau

🌞 **Vous me prouverez que :**

- le service ssh est actif sur la VM
  - avec une commande `systemctl status ...`
  ```
  [marceau@localhost ~]$ systemctl status
    ● localhost.localdomain
        State: running
         Jobs: 0 queued
       Failed: 0 units
        Since: Wed 2021-11-24 21:49:15 CET; 4min 2s ago
    ```
- vous pouvez vous connecter à la VM, grâce à un échange de clés
  - référez-vous [au cours sur SSH pour + de détails sur l'échange de clés](../../cours/cours/SSH/README.md)

> Pour me prouver que vous pouvez vous connecter avec un échange de clé il faut me montrer, dans l'idéal : la clé publique (le cadenas) sur votre PC (un `cat` du fichier), un `cat` du fichier `authorized_keys` concerné sur la VM, et une connexion sans aucun mot de passe demandé.

---

➜ **Accès internet**

Par l'expression commune et barbare "avoir un accès internet" on entend deux choses généralement :

- *ahem* avoir un accès internet
  - c'est à dire être en mesure de ping des IP publiques
- avoir de la résolution de noms
  - la machine doit pouvoir traduire un nom comme `google.com` vers l'IP associée

🌞 **Prouvez que vous avez un accès internet**

- avec une commande `ping`

> On utilise souvent un `ping` vers une adresse IP publique connue pour tester l'accès internet.  
Vous verrez souvent `8.8.8.8` c'est l'adresse d'un serveur Google. On part du principe que Google sera bien le dernier à quitter Internet, donc on teste l'accès internet en le pingant.  
Si vous n'aimez pas Google comme moi, vous pouvez `ping 1.1.1.1`, c'est les serveurs de CloudFlare.
```
[marceau@localhost ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=113 time=20.5 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=113 time=24.4 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=113 time=23.5 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=113 time=20.9 ms
64 bytes from 8.8.8.8: icmp_seq=5 ttl=113 time=26.4 ms
```

🌞 **Prouvez que vous avez de la résolution de nom**

Un petit `ping` vers un nom de domaine, celui que vous voulez :)
```
[marceau@localhost ~]$ ping google.com
PING google.com (216.58.214.174) 56(84) bytes of data.
64 bytes from mad01s26-in-f174.1e100.net (216.58.214.174): icmp_seq=1 ttl=116 time=27.2 ms
64 bytes from mad01s26-in-f174.1e100.net (216.58.214.174): icmp_seq=2 ttl=116 time=24.6 ms
64 bytes from mad01s26-in-f174.1e100.net (216.58.214.174): icmp_seq=3 ttl=116 time=24.6 ms
64 bytes from mad01s26-in-f174.1e100.net (216.58.214.174): icmp_seq=4 ttl=116 time=34.5 ms
64 bytes from mad01s26-in-f174.1e100.net (216.58.214.174): icmp_seq=5 ttl=116 time=23.7 ms
````
---

➜ **Nommage de la machine**

Votre chambre vous la rangez na ? Au moins de temps en temps ? Là c'est pareil : on range les machines. Et ça commence par leur donner un nom.

Pour donner un nom à une machine il faut exécuter deux actions :

- changer son nom immédiatement, mais le changement sera perdu au reboot
- écrire son nom dans un fichier, pour que le fichier soit lu au boot et que le changement persiste

En commande ça donne :

```bash
# Pour la session
$ sudo hostname <NOM>

# Persistant après les reboots
$ sudo nano /etc/hostname # remplacer le contenu du fichier par le NOM
```

🌞 **Définissez `node1.tp4.linux` comme nom à la machine**

- montrez moi le contenu du fichier `/etc/hostname`
```
[marceau@localhost ~]$ cat /etc/hostname
node1.tp4.linux
```
- tapez la commande `hostname` (sans argument ni option) pour afficher votre hostname actuel
```
[marceau@localhost ~]$ hostname
node1.tp4.linux
```

> Vous verrez aussi votre hostname dans le prompt du terminal, en plus de l'utilisateur avec lequel vous êtes co : `it4@node1:~$`.  
C'est de l'anglais en fait hein, le "@" se dit "at". Donc en l'occurence "it4 at node1".

# III. Mettre en place un service

On va installer un service de test pour voir la procédure récurrente d'installation et configuration d'un nouveau service.  
C'est un peu la routine de l'administrateur système. My job heh.

Toujours les mêmes opérations :

- **install du service**
  - souvent via des paquets
- puis **conf du service**
  - dans des fichiers texte de configuration
- puis si c'est un service qui utilise le réseau, il faudra **ouvrir** un port dans le pare-feu
  - hé ui, fini les distribs comme xubuntu où c'est open-bar
  - Rocky Linux bloque la plupart du trafic qui essaie d'entrer sur la machine
- puis **lancement du service**
  - ptite commande `systemctl start` généralement
- puis **enjoy**
  - consommation du service par des clients

On va setup un serveur web. Cas d'école un peu, on va s'en servir pour approfondir encore notre maîtrise des services.

## 1. Intro NGINX

![gnignigggnnninx ?](./pics/ngnggngngggninx.jpg)

NGINX (prononcé "engine-X") est un serveur web. C'est un outil de référence aujourd'hui, il est réputé pour ses performances et sa robustesse.

Ici on va pas DU TOUT s'attarder sur la partie dév web étou, une simple page HTML fera l'affaire.

Une fois le serveur web installé, on récupère :

- **un service**
  - un service c'est un processus
  - il y a donc un binaire, une application qui fait serveur web
  - qui dit processus, dit que quelqu'un, un utilisateur lance ce processus
  - c'est l'utilisateur qu'on voit lister dans la sortie de `ps -ef`
- **des fichiers de conf**
  - comme d'hab c'est dans `/etc/` la conf
  - comme d'hab c'est bien rangé, donc la conf de NGINX c'est dans `/etc/nginx/`
  - question de simplicité en terme de nommage, le fichier de conf principal c'est `/etc/nginx/nginx.conf`
  - la conf, ça appartient à l'utilisateur `root`
- **une racine web**
  - c'est un dossier dans lequel le site est stocké
  - c'est à dire là où se trouvent tous les fichiers PHP, HTML, CSS, JS, etc du site
  - ce dossier et tout son contenu doivent appartenir à l'utilisateur qui lance le service
- **des logs**
  - tant que le service a pas trop tourné c'est empty
  - comme d'hab c'est `/var/log/`
  - comme d'hab c'est bien rangé donc c'est dans `/var/log/nginx/`
  - comme d'hab on peut aussi consulter certains logs avec `sudo journalctl -xe -u nginx`

## 2. Install

🌞 **Installez NGINX en vous référant à des docs online**

Vous devez comprendre toutes les commandes que vous tapez. En deux trois commandes c'est plié l'install.

```
[marceau@node1 ~]$ sudo dnf install nginx

[marceau@node1 ~]$ sudo systemctl start nginx

[marceau@node1 ~]$ systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2021-11-24 22:51:21 CET; 13s ago
  Process: 4147 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 4146 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 4144 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 4149 (nginx)
    Tasks: 2 (limit: 4956)
   Memory: 3.7M
```

## 3. Analyse

Avant de config étou, on va lancer à l'aveugle et inspecter ce qu'il se passe.

Commencez donc par démarrer le service NGINX :

```bash
$ sudo systemctl start nginx
$ sudo systemctl status nginx
```

🌞 **Analysez le service NGINX**

- avec une commande `ps`, déterminer sous quel utilisateur tourne le processus du service NGINX
```
[marceau@node1 ~]$ sudo ps -ef | grep nginx
[sudo] password for marceau:
root        4149       1  0 22:51 ?        00:00:00 nginx: master process /usr/sbin/nginx
nginx       4150    4149  0 22:51 ?        00:00:00 nginx: worker process
marceau     4174    1508  0 23:07 pts/0    00:00:00 grep --color=auto nginx
```
- avec une commande `ss`, déterminer derrière quel port écoute actuellement le serveur web
```
[marceau@node1 ~]$ sudo ss -ltpn | grep nginx
LISTEN 0      128          0.0.0.0:80        0.0.0.0:*    users:(("nginx",pid=1542,fd=8),("nginx",pid=1541,fd=8))
LISTEN 0      128             [::]:80           [::]:*    users:(("nginx",pid=1542,fd=9),("nginx",pid=1541,fd=9))
```
- en regardant la conf, déterminer dans quel dossier se trouve la racine web
```
[marceau@node1 ~]$ sudo cat /etc/nginx/nginx.conf
        root         /usr/share/nginx/html;
```
- inspectez les fichiers de la racine web, et vérifier qu'ils sont bien accessibles en lecture par l'utilisateur qui lance le processus
```
[marceau@node1 ~]$ ls -la /usr/share/nginx/html/
total 20
drwxr-xr-x. 2 root root   99 23 nov.  17:07 .
drwxr-xr-x. 4 root root   33 23 nov.  17:07 ..
-rw-r--r--. 1 root root 3332 10 juin  05:09 404.html
-rw-r--r--. 1 root root 3404 10 juin  05:09 50x.html
-rw-r--r--. 1 root root 3429 10 juin  05:09 index.html
-rw-r--r--. 1 root root  368 10 juin  05:09 nginx-logo.png
-rw-r--r--. 1 root root 1800 10 juin  05:09 poweredby.pn`

```

## 4. Visite du service web

Et ça serait bien d'accéder au service non ? Bon je vous laisse pas dans le mur : spoiler alert, le service est actif, mais le firewall de Rocky bloque l'accès au service, on va donc devoir le configurer.

Il existe [un mémo dédié au réseau au réseau sous Rocky](../../cours/memos/rocky_network.md), vous trouverez le nécessaire là-bas pour le firewall.

🌞 **Configurez le firewall pour autoriser le trafic vers le service NGINX** (c'est du TCP ;) )
```
[marceau@node1 ~]$ sudo firewall-cmd --add-port=80/tcp --permanent
[marceau@node1 ~]$ sudo firewall-cmd --reload
```

🌞 **Tester le bon fonctionnement du service**

- avec votre navigateur sur VOTRE PC
  - ouvrez le navigateur vers l'URL : `http://<IP_VM>:<PORT>`
- vous pouvez aussi effectuer des requêtes HTTP depuis le terminal, plutôt qu'avec un navigateur
  - ça se fait avec la commande `curl`
  - et c'est ça que je veux dans le compte-rendu, pas de screen du navigateur :)

```
[marceau@node1 ~]$ curl http://10.200.1.10
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">

<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en">
[...]
```

## 5. Modif de la conf du serveur web

🌞 **Changer le port d'écoute**

- une simple ligne à modifier, vous me la montrerez dans le compte rendu
  - faites écouter NGINX sur le port 8080
- redémarrer le service pour que le changement prenne effet
  - `sudo systemctl restart nginx`
  - vérifiez qu'il tourne toujours avec un ptit `systemctl status nginx`
- prouvez-moi que le changement a pris effet avec une commande `ss`
- n'oubliez pas de fermer l'ancier port dans le firewall, et d'ouvrir le nouveau
- prouvez avec une commande `curl` sur votre machine que vous pouvez désormais visiter le port 8080

---

```
[marceau@node1 ~]$ sudo firewall-cmd --remove-port=80/tcp --permanent
[marceau@node1 ~]$ sudo firewall-cmd --add-port=8080/tcp --permanent
[marceau@node1 ~]$ sudo firewall-cmd --reload
[marceau@node1 ~]$ sudo systemctl restart nginx
[marceau@node1 ~]$ sudo systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor pres>
   Active: active (running) since Thur 2021-11-25 18:42:32 EST; 9s ago

[marceau@node1 ~]$ sudo ss -ltpn | grep nginx
LISTEN 0      128          0.0.0.0:8080      0.0.0.0:*    users:(("nginx",pid=4747,fd=8),("nginx",pid=4746,fd=8))
LISTEN 0      128             [::]:8080         [::]:*    users:(("nginx",pid=4747,fd=9),("nginx",pid=4746,fd=9))

```

🌞 **Changer l'utilisateur qui lance le service**

- pour ça, vous créerez vous-même un nouvel utilisateur sur le système : `web`
  - référez-vous au [mémo des commandes](../../cours/memos/commandes.md) pour la création d'utilisateur
  - l'utilisateur devra avoir un mot de passe, et un homedir défini explicitement à `/home/web`
```
[marceau@node1 ~]$ sudo useradd web -m -s /bin/sh -u 2000
[marceau@node1 ~]$ sudo passwd web
Changement de mot de passe pour l'utilisateur web.
Nouveau mot de passe : 
passwd : mise à jour réussie de tous les jetons d'authentification
```
- un peu de conf à modifier dans le fichier de conf de NGINX pour définir le nouvel utilisateur en tant que celui qui lance le service
  - vous me montrerez la conf effectuée dans le compte-rendu
- n'oubliez pas de redémarrer le service pour que le changement prenne effet
- vous prouverez avec une commande `ps` que le service tourne bien sous ce nouveau utilisateur
```
[marceau@node1 ~]$ ps -ef | grep nginx
root        4859       1  0 18:50 ?        00:00:00 nginx: master process /usr/sbin/nginx
web         4860    4859  0 18:50 ?        00:00:00 nginx: worker process
marceau     4865    2004  0 18:51 pts/0    00:00:00 grep --color=auto nginx
```

---

🌞 **Changer l'emplacement de la racine Web**

- vous créerez un nouveau dossier : `/var/www/super_site_web`
  - avec un fichier  `/var/www/super_site_web/index.html` qui contient deux trois lignes de HTML, peu importe, un bon `<h1>toto</h1>` des familles, ça fera l'affaire
  - le dossier et tout son contenu doivent appartenir à `web`
- configurez NGINX pour qu'il utilise cette nouvelle racine web
  - vous me montrerez la conf effectuée dans le compte-rendu
- n'oubliez pas de redémarrer le service pour que le changement prenne effet
- prouvez avec un `curl` depuis votre hôte que vous accédez bien au nouveau site

```
[marceau@node1 ~]$ sudo mkdir /var/www
[marceau@node1 ~]$ sudo chmod -R 777 /var/www/
[marceau@node1 ~]$ sudo su - web
[web@node1 ~]$ cd /var/www/
[web@node1 www]$  mkdir super_site_web
[web@node1 www]$ nano index.html
[web@node1 www]$ ls -al
total 4
drwxrwxrwx.  3 root root   28 Nov 23 12:44 .
drwxr-xr-x. 22 root root 4096 Nov 23 12:38 ..
drwxrwxr-x.  2 web  web    24 Nov 23 12:45 super_site_web
[web@node1 www]$ curl 10.210.1.8:8080
<!DOCTYPE html>
<p>Oui</p>
```