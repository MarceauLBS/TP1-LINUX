# TP5 : P'tit cloud perso

# I. Setup DB

Côté base de données, on va utiliser MariaDB.

## Sommaire

- [I. Setup DB](#i-setup-db)
  - [Sommaire](#sommaire)
  - [1. Install MariaDB](#1-install-mariadb)
  - [2. Conf MariaDB](#2-conf-mariadb)
  - [3. Test](#3-test)

## 1. Install MariaDB

> Pour rappel, le gestionnaire de paquets sous les OS de la famille RedHat, c'est pas `apt`, c'est `dnf`.

🌞 **Installer MariaDB sur la machine `db.tp5.linux`**

- le paquet s'appelle `mariadb-server`
- le service s'appelle `mariadb`
```
[marceau@node1 ~]$ sudo dnf install mariadb-server
[...]
```

🌞 **Le service MariaDB**

- lancez-le avec une commande `systemctl`
```
[marceau@node1 ~]$ systemctl start mariadb
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ====
Authentication is required to start 'mariadb.service'.
Authenticating as: marceau
Password:
==== AUTHENTICATION COMPLETE ====
```
- exécutez la commande `sudo systemctl enable mariadb` pour faire en sorte que MariaDB se lance au démarrage de la machine
```
[marceau@node1 ~]$ sudo systemctl enable mariadb
[sudo] password for marceau:
Created symlink /etc/systemd/system/mysql.service → /usr/lib/systemd/system/mariadb.service.
Created symlink /etc/systemd/system/mysqld.service → /usr/lib/systemd/system/mariadb.service.
Created symlink /etc/systemd/system/multi-user.target.wants/mariadb.service → /usr/lib/systemd/system/mariadb.service.
```
- vérifiez qu'il est bien actif avec une commande `systemctl`
```
[marceau@node1 ~]$ systemctl status mariadb
● mariadb.service - MariaDB 10.3 database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; disabled; vendor preset: disabled)
   Active: active (running) since Thu 2021-11-25 11:57:33 CET; 2min 21s ago
```
- déterminer sur quel port la base de données écoute avec une commande `ss`
  - je veux que l'information soit claire : le numéro de port avec le processus qu'il y a derrière
```
[marceau@node1 ~]$ sudo ss -lntp | grep sql
LISTEN 0      80                 *:3306            *:*    users:(("mysqld",pid=4666,fd=21))
````
- isolez les processus liés au service MariaDB (commande `ps`)
  - déterminez sous quel utilisateur est lancé le process MariaDB
```
[marceau@db ~]$ ps -ef | grep sql
mysql        955       1  0 09:43 ?        00:00:00 /usr/libexec/mysqld --basedir=/usr
marceau     1654    1627  0 09:54 pts/0    00:00:00 grep --color=auto sql
```

🌞 **Firewall**

- pour autoriser les connexions qui viendront de la machine `web.tp5.linux`, il faut conf le firewall
  - ouvrez le port utilisé par MySQL à l'aide d'une commande `firewall-cmd`
```
[marceau@db ~]$ sudo firewall-cmd --add-port=3306/tcp --permanent
[sudo] password for marceau:
success
```

## 2. Conf MariaDB

Première étape : le `mysql_secure_installation`. C'est un binaire qui sert à effectuer des configurations très récurrentes, on fait ça sur toutes les bases de données à l'install.  
C'est une question de sécu.

🌞 **Configuration élémentaire de la base**

- exécutez la commande `mysql_secure_installation`
  - plusieurs questions successives vont vous être posées
  - expliquez avec des mots, de façon concise, ce que signifie chacune des questions
  - expliquez pourquoi vous répondez telle ou telle réponse (avec la sécurité en tête)

On me demande d'abord si un mdp est déjà mis en place pour root :
```
- In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.
```
Je réponds donc que non et on me demande d'en créer un nouveau.
Une fois le MDP créé, on me demande si je veux supprimer le fait que tout le monde peut se connecter à MariaDB :
```
- By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.
```
(Bien sur que je réponds oui)
Ensuite, on me demande si seule la machine originelle nous permet de se connecter en route :
```
- Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.
```
(Encore oui)
On me demande si je veux Supprimer la base de donnée "test" que n'importe qui peux accéder :
```
- By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.
```
(Oui oui oui oui...)
Pour finir on me demande si je souhaite que les changements éffectués doivent être immédiatement appliqués ou non :
```
- Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.
```
On répond encore oui.

---

🌞 **Préparation de la base en vue de l'utilisation par NextCloud**

- pour ça, il faut vous connecter à la base
- il existe un utilisateur `root` dans la base de données, qui a tous les droits sur la base
- si vous avez correctement répondu aux questions de `mysql_secure_installation`, vous ne pouvez utiliser le user `root` de la base de données qu'en vous connectant localement à la base
- donc, sur la VM `db.tp5.linux` toujours :

Puis, dans l'invite de commande SQL :

```
[marceau@db ~]$ sudo mysql -u root -p
CREATE USER 'nextcloud'@'10.5.1.11' IDENTIFIED BY 'meow';
CREATE DATABASE IF NOT EXISTS nextcloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'10.5.1.11';
FLUSH PRIVILEGES;
EXIT
bye
[marceau@db ~]$ 
```
## 3. Test

Bon, là il faut tester que la base sera utilisable par NextCloud.

Concrètement il va faire quoi NextCloud vis-à-vis de la base MariaDB ?

- se connecter sur le port où écoute MariaDB
- la connexion viendra de `web.tp5.linux`
- il se connectera en utilisant l'utilisateur `nextcloud`
- il écrira/lira des données dans la base `nextcloud`

Il faudrait donc qu'on teste ça, à la main, depuis la machine `web.tp5.linux`.

Bah c'est parti ! Il nous faut juste un client pour nous connecter à la base depuis la ligne du commande : il existe une commande `mysql` pour ça.

🌞 **Installez sur la machine `web.tp5.linux` la commande `mysql`**

- vous utiliserez la commande `dnf provides` pour trouver dans quel paquet se trouve cette commande

```
[marceau@web ~]$ dnf provides mysql
mysql-8.0.26-1.module+el8.4.0+652+6de068a7.x86_64
[marceau@web ~]$ sudo dnf install 8.0.26-1.module+el8.4.0+652+6de068a7
[sudo] password for marceau:
Installed:
  mariadb-connector-c-config-3.1.11-2.el8_3.noarch
  mysql-8.0.26-1.module+el8.4.0+652+6de068a7.x86_64
  mysql-common-8.0.26-1.module+el8.4.0+652+6de068a7.x86_64

Complete!
```

🌞 **Tester la connexion**

- utilisez la commande `mysql` depuis `web.tp5.linux` pour vous connecter à la base qui tourne sur `db.tp5.linux`
- vous devrez préciser une option pour chacun des points suivants :
  - l'adresse IP de la machine où vous voulez vous connectez `db.tp5.linux` : `10.5.1.12`
  - le port auquel vous vous connectez
  - l'utilisateur de la base de données sur lequel vous connecter : `nextcloud`
  - l'option `-p` pour indiquer que vous préciserez un password
    - vous ne devez PAS le préciser sur la ligne de commande
    - sinon il y aurait un mot de passe en clair dans votre historique, c'est moche
  - la base de données à laquelle vous vous connectez : `nextcloud`
- une fois connecté à la base en tant que l'utilisateur `nextcloud` :
  - effectuez un bête `SHOW TABLES;`
  - simplement pour vous assurer que vous avez les droits de lecture
  - et constater que la base est actuellement vide

```
[marceau@web ~]$ mysql -h 10.5.1.12 -P 3306 -u nextcloud -p nextcloud
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
[...]
MariaDB [nextcloud]> SHOW TABLES;
Empty set (0.001 sec)
````

# II. Setup Web



## Sommaire

- [II. Setup Web](#ii-setup-web)
  - [Sommaire](#sommaire)
  - [1. Install Apache](#1-install-apache)
    - [A. Apache](#a-apache)
    - [B. PHP](#b-php)
  - [2. Conf Apache](#2-conf-apache)
  - [3. Install NextCloud](#3-install-nextcloud)
  - [4. Test](#4-test)

## 1. Install Apache

### A. Apache

🌞 **Installer Apache sur la machine `web.tp5.linux`**

- le paquet qui contient Apache s'appelle `httpd`
```
[marceau@db ~]$ systemctl start httpd
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ====
Authentication is required to start 'httpd.service'.
Authenticating as: marceau
Password:
==== AUTHENTICATION COMPLETE ====
```
- le service aussi s'appelle `httpd`
```
[marceau@db ~]$ systemctl status httpd
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
   Active: active (running) since Mon 2021-11-29 19:30:13 CET; 12s ago
     Docs: man:httpd.service(8)
 Main PID: 2208 (httpd)
```
---

🌞 **Analyse du service Apache**

````
[marceau@db ~]$ sudo systemctl enable httpd
Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.
[marceau@db ~]$ sudo ss -ltpn | grep httpd
LISTEN 0      128                *:80              *:*    users:(("httpd",pid=2212,fd=4),("httpd",pid=2211,fd=4),("httpd",pid=2210,fd=4),("httpd",pid=2208,fd=4))
[marceau@db ~]$ sudo ps -ef | grep httpd
root        2208       1  0 19:30 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache      2209    2208  0 19:30 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache      2210    2208  0 19:30 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache      2211    2208  0 19:30 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache      2212    2208  0 19:30 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
marceau     2440    1642  0 19:43 pts/0    00:00:00 grep --color=auto httpd
````

---

🌞 **Un premier test**
```
[marceau@db ~]$ sudo firewall-cmd --add-port=80/tcp --permanent
[sudo] password for marceau:
success
[marceau@db ~]$ sudo firewall-cmd --reload
success
```

### B. PHP

NextCloud a besoin d'une version bien spécifique de PHP.  
Suivez **scrupuleusement** les instructions qui suivent pour l'installer.

🌞 **Installer PHP**

```bash

[marceau@db ~]$ sudo dnf install epel-release
[...]
Complete!
[marceau@db ~]$ sudo dnf update
[...]
Complete!
[marceau@db ~]$ sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-8.rpm
[marceau@db ~]$ sudo dnf module enable php:remi-7.4
[marceau@db ~]$ sudo dnf install zip unzip libxml2 openssl php74-php php74-php-ctype php74-php-curl php74-php-gd php74-php-iconv php74-php-json php74-php-libxml php74-php-mbstring php74-php-openssl php74-php-posix php74-php-session php74-php-xml php74-php-zip php74-php-zlib php74-php-pdo php74-php-mysqlnd php74-php-intl php74-php-bcmath php74-php-gmp
[...]
Complete!
```

## 2. Conf Apache

➜ Le fichier de conf utilisé par Apache est `/etc/httpd/conf/httpd.conf`.  
Il y en a plein d'autres : ils sont inclus par le premier.

➜ Dans Apache, il existe la notion de *VirtualHost*. On définit des *VirtualHost* dans les fichiers de conf d'Apache.  
On crée un *VirtualHost* pour chaque application web qu'héberge Apache.

> "Application Web" c'est le terme de hipster pour désigner un site web. Disons qu'aujourd'hui les sites peuvent faire tellement de trucs qu'on appelle plutôt ça une "application" à part entière. Une application web donc.

➜ Dans le dossier `/etc/httpd/` se trouve un dossier `conf.d`.  
Des dossiers qui se terminent par `.d`, vous en rencontrerez plein, ce sont des dossiers de *drop-in*.  
Plutôt que d'écrire 40000 lignes dans un seul fichier de conf, on l'éclate en plusieurs fichiers la conf.  
C'est + lisible et + facilement maintenable.

Les dossiers de *drop-in* servent à accueillir ces fichiers de conf additionels.  
Le fichier de conf principal a une ligne qui inclut tous les fichiers de conf contenus dans le dossier de *drop-in*.

---

🌞 **Analyser la conf Apache**

- mettez en évidence, dans le fichier de conf principal d'Apache, la ligne qui inclut tout ce qu'il y a dans le dossier de *drop-in*
```
[marceau@db ~]$ cat /etc/httpd/conf/httpd.conf
[...]
IncludeOptional conf.d/*.conf
```

🌞 **Créer un VirtualHost qui accueillera NextCloud**

- créez un nouveau fichier dans le dossier de *drop-in*
  - attention, il devra être correctement nommé (l'extension) pour être inclus par le fichier de conf principal
- ce fichier devra avoir le contenu suivant :

```apache
[marceau@web ~]$ cat my_website.conf 
<VirtualHost *:80>
  DocumentRoot /var/www/nextcloud/html/
  ServerName  web.tp5.linux

  <Directory /var/www/nextcloud/html/>
    Require all granted
    AllowOverride All
    Options FollowSymLinks MultiViews

    <IfModule mod_dav.c>
      Dav off
    </IfModule>
  </Directory>
</VirtualHost>
```

> N'oubliez pas de redémarrer le service à chaque changement de la configuration, pour que les changements prennent effet.

🌞 **Configurer la racine web**

- la racine Web, on l'a configurée dans Apache pour être le dossier `/var/www/nextcloud/html/`
- creéz ce dossier
- faites appartenir le dossier et son contenu à l'utilisateur qui lance Apache (commande `chown`, voir le [mémo commandes](../../cours/memos/commandes.md))

````
[marceau@web ~]$ sudo mkdir -p /var/www/nextcloud/html/
[marceau@web ~]$ sudo chown -R apache:apache /var/www
````

🌞 **Configurer PHP**

- dans l'install de NextCloud, PHP a besoin de conaître votre timezone (fuseau horaire)
- pour récupérer la timezone actuelle de la machine, utilisez la commande `timedatectl` (sans argument)
- modifiez le fichier `/etc/opt/remi/php74/php.ini` :
  - changez la ligne `;date.timezone =`
  - par `date.timezone = "<VOTRE_TIMEZONE>"`
  - par exemple `date.timezone = "Europe/Paris"`
```
[marceau@web ~]$ cat /etc/opt/remi/php74/php.ini | grep Paris
date.timezone = "Europe/Paris"
```

## 3. Install NextCloud

On dit "installer NextCloud" mais en fait c'est juste récupérer les fichiers PHP, HTML, JS, etc... qui constituent NextCloud, et les mettre dans le dossier de la racine web.

🌞 **Récupérer Nextcloud**

```bash=
[marceau@web www]$ cd
[marceau@web ~]$ curl -SLO https://download.nextcloud.com/server/releases/nextcloud-21.0.1.zip
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  148M  100  148M    0     0  13.7M      0  0:00:10  0:00:10 --:--:-- 15.1M
[marceau@web ~]$ ls
nextcloud-21.0.1.zip
```

🌞 **Ranger la chambre**

- extraire le contenu de NextCloud (beh ui on a récup un `.zip`)
- déplacer tout le contenu dans la racine Web
  - n'oubliez pas de gérer les permissions de tous les fichiers déplacés ;)
- supprimer l'archive
```
[marceau@web ~]$ unzip nextcloud-21.0.1.zip
[...]
[marceau@web ~]$ ls
nextcloud  nextcloud-21.0.1.zip

```

## 4. Test

🌞 **Modifiez le fichier `hosts` de votre PC**

- ajoutez la ligne : `10.5.1.11 web.tp5.linux`

```
[marceau@web ~]$ sudo vim /etc/hosts
# Host addresses
127.0.0.1  localhost
127.0.1.1  node1
::1        localhost ip6-localhost ip6-loopback
ff02::1    ip6-allnodes
ff02::2    ip6-allrouters
10.5.1.11 web.tp5.linux
```

🌞 **Tester l'accès à NextCloud et finaliser son install'**

- ouvrez votre navigateur Web sur votre PC
- rdv à l'URL `http://web.tp5.linux`
- vous devriez avoir la page d'accueil de NextCloud
- ici deux choses :
  - les deux champs en haut pour créer un user admin au sein de NextCloud
  - le bouton "Configure the database" en bas
    - sélectionnez "MySQL/MariaDB"
    - entrez les infos pour que NextCloud sache comment se connecter à votre serveur de base de données
    - c'est les infos avec lesquelles vous avez validé à la main le bon fonctionnement de MariaDB (c'était avec la commande `mysql`)

```
[marceau@web ~]$ curl http://web.tp5.linux -L | head -n 4
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
<!DOCTYPE html>
<html class="ng-csp" data-placeholder-focus="false" lang="en" data-locale="en" >
	<head
 data-requesttoken="0ZiNy7+nngHwoHwoLZvyQl2vvfGmhe7CORsWaqEnXBE=:v+i6ue/k/DeU6j1rWvmmOw/7/5fS17qTdV1aPJN1O3g=">
100  9973  100  9973    0     0  77600      0 --:--:-- --:--:-- --:--:-- 77600
```