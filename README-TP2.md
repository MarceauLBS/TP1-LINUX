# **TP2 : Explorer et manipuler le système**

## **Prérequis**

### -> Changer le nom de la machine : 

--------------------------
- **Première étape :** changer le nom tout de suite, **jusqu'à ce qu'on redémarre la machine**

    Nous allons tapper cette commande : ```sudo hostname <NOM_MACHINE>```
    ```bash 
    marceau@marceau-vm:~$ sudo hostname node1.tp2.linux
    ```
    Elle permet de changer **le nom de notre machine** en "node1" jusqu'au **prochain reboot** de celle-ci.
    Nous pouvons vérifier si le changement a été effectué, nous pouvons tapper ceci : ```hostname```
    ```bash
    marceau@marceau-vm:~$ hostname
    node1.tp2.linux
    ```
    Nous voyons ici que le changement de nom **a été pris en compte**.

- **Deuxième étape :** changer le nom qui est pris par la machine quand elle s'allume

    Nous allons maintenant modifier le dossier /home/hostname afin de changer le nom de la machine meme après un reboot.
    Pour ce faire, nous utilisons la commande : ```sudo nano /etc/hostname```
    ```bash
    marceau@marceau-vm:~$ sudo nano /etc/hostname
    ```
    Une fois dans le fichier texte, nous changeons le nom de la machine par "node1.tp2.linux".
    
    Comme précédemment, nous allons vérifier si le changement a été compris par la machine : 
    ```bash
    marceau@marceau-vm:~$ cat /etc/hostname
    node1.tp2.linux
    ```
    
---
### -> Config réseau fonctionnelle

Nous allons maintenant vérifier que notre VM est **connectée à internet.**
    
Pour ce faire, nous allons tester **plusieurs méthodes** :
    
- **La 1ère :** une connexion à l'adresse ip d'un serveur dns de google (8.8.8.8)
        
```bash
    marceau@marceau-vm:~$ ping 8.8.8.8
    PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
    64 bytes from 8.8.8.8: icmp_seq=1 ttl=113 time=25.8 ms
    64 bytes from 8.8.8.8: icmp_seq=2 ttl=113 time=24.0 ms
    64 bytes from 8.8.8.8: icmp_seq=3 ttl=113 time=25.1 ms
```
Nous voyons que la connexion est réussie avec ce serveur.
    
- **La 2nde :** une connexion au site ynov.com
    
```bash
    marceau@marceau-vm:~$ ping ynov.com
    PING ynov.com (92.243.16.143) 56(84) bytes of data.
    64 bytes from xvm-16-143.dc0.ghst.net (92.243.16.143): icmp_seq=1 ttl=50 time=21.4 ms
    64 bytes from xvm-16-143.dc0.ghst.net (92.243.16.143): icmp_seq=2 ttl=50 time=23.8 ms
    64 bytes from xvm-16-143.dc0.ghst.net (92.243.16.143): icmp_seq=3 ttl=50 time=21.8 ms
```
Encore une fois, la connexion avec le domaine est réussie.
    
- **La dernière :** une connexion à la VM depuis mon pc
    
```bash
    C:\Users\marce>ping 192.168.56.117

    Envoi d’une requête 'Ping'  192.168.56.117 avec 32 octets de données :
    Réponse de 192.168.56.117 : octets=32 temps<1ms TTL=64
    Réponse de 192.168.56.117 : octets=32 temps<1ms TTL=64
    Réponse de 192.168.56.117 : octets=32 temps<1ms TTL=64
    Réponse de 192.168.56.117 : octets=32 temps<1ms TTL=64

    Statistiques Ping pour 192.168.56.117:
        Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
    Durée approximative des boucles en millisecondes :
        Minimum = 0ms, Maximum = 0ms, Moyenne = 0ms
````
---

# Partie 1 : SSH

# I. Intro

> *SSH* c'est pour *Secure SHell*.

***SSH* est un outil qui permet d'accéder au terminal d'une machine, à distance, en connaissant l'adresse IP de cette machine.**

*SSH* repose un principe de *client/serveur* :

- le *serveur*
  - est installé sur une machine par l'admin
  - écoute sur le port 22/TCP par convention
- le *client*
  - connaît l'IP du serveur
  - se connecte au serveur à l'aide d'un programme appelé "*client* *SSH*"

Pour nous, ça va être l'outil parfait pour contrôler toutes nos machines virtuelles.

Dans la vie réelle, *SSH* est systématiquement utilisé pour contrôler des machines à distance. C'est vraiment un outil de routine pour tout informaticien, qu'on utilise au quotidien sans y réfléchir.

> *Si vous louez un serveur en ligne, on vous donnera un accès SSH pour le manipuler la plupart du temps.*

# II. Setup du serveur SSH

Toujours la même routine :

- **1. installation de paquet**
  - avec le gestionnaire de paquet de l'OS
- **2. configuration** dans un fichier de configuration
  - avec un éditeur de texte
  - les fichiers de conf sont de simples fichiers texte
- **3. lancement du service**
  - avec une commande `systemctl start <NOM_SERVICE>`

> ***Pour toutes les commandes tapées qui figurent dans le rendu, je veux la commande ET son résultat. S'il manque l'un des deux, c'est useless.***

## 1. Installation du serveur

Sur les OS GNU/Linux, les installations se font à l'aide d'un gestionnaire de paquets.

🌞 **Installer le paquet `openssh-server`**

- avec une commande `apt install`

```bash=
marceau@node1:~$ sudo -i
[sudo] password for marceau:
root@node1:~# apt install openssh-server
Reading package lists... Done
Building dependency tree
Reading state information... Done
openssh-server is already the newest version (1:8.2p1-4ubuntu0.3).
0 upgraded, 0 newly installed, 0 to remove and 61 not upgraded.
```

---

Note : une fois que le paquet est installé, plusieurs nouvelles choses sont dispos sur la machine. Notamment :

- un service `ssh`
- un dossier de configuration `/etc/ssh/`

## 2. Lancement du service SSH

🌞 **Lancer le service `ssh`**

- avec une commande `systemctl start`
- vérifier que le service est actuellement actif avec une commande `systemctl status`

```bash=
root@node1:/# systemctl start ssh
root@node1:/# systemctl status ssh
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2021-10-30 20:28:00 CEST; 1 weeks 0 days ago
       Docs: man:sshd(8)
             man:sshd_config(5)
   Main PID: 565 (sshd)
      Tasks: 1 (limit: 2312)
     Memory: 6.6M
     CGroup: /system.slice/ssh.service
             └─565 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups

oct. 30 20:28:00 node1.tp2.linux systemd[1]: Starting OpenBSD Secure Shell server...
oct. 30 20:28:00 node1.tp2.linux sshd[565]: Server listening on 0.0.0.0 port 22.
oct. 30 20:28:00 node1.tp2.linux sshd[565]: Server listening on :: port 22.
oct. 30 20:28:00 node1.tp2.linux systemd[1]: Started OpenBSD Secure Shell server.
oct. 30 20:39:35 node1.tp2.linux sshd[1366]: Accepted password for marceau from 192.168.56.1 port 62894 ssh2
oct. 30 20:39:35 node1.tp2.linux sshd[1366]: pam_unix(sshd:session): session opened for user marceau by (uid=0)
nov. 07 17:00:15 node1.tp2.linux sshd[26555]: Accepted password for marceau from 192.168.56.1 port 49875 ssh2
nov. 07 17:00:15 node1.tp2.linux sshd[26555]: pam_unix(sshd:session): session opened for user marceau by (uid=0)
```

> Vous pouvez aussi faire en sorte que le *service* SSH se lance automatiquement au démarrage avec la commande `systemctl enable ssh`.

## 3. Etude du service SSH

🌞 **Analyser le service en cours de fonctionnement**

- afficher le statut du *service*
  - avec une commande `systemctl status`
- afficher le/les processus liés au *service* `ssh`
  - avec une commande `ps`
  - isolez uniquement la/les ligne(s) intéressante(s) pour le rendu de TP
  ```bash
  root@node1:/# ps aux
    root        1366  0.0  0.3  14024  7680 ?        Ss   16:36   0:00     sshd:         marceau [priv]
    marceau     1447  0.0  0.2  14024  4364 ?        S    16:36   0:00 sshd:     marceau@pts/1
    ```
- afficher le port utilisé par le *service* `ssh`
  - avec une commande `ss -l`
  - isolez uniquement la/les ligne(s) intéressante(s)
  ```bash
  root@node1:/# ss -l
  tcp   LISTEN 0      128                                          [::]:ssh                            [::]:*
  ```
- afficher les logs du *service* `ssh`
  - avec une commande `journalctl`
  - en consultant un dossier dans `/var/log/`
  - ne me donnez pas toutes les lignes de logs, je veux simplement que vous appreniez à consulter les logs
  ```bash
  root@node1:/# journalctl | grep sshd
  oct. 19 12:39:00 marceau-vm    useradd[1436]: new user: name=sshd, UID=123, GID=65534, home=/run/sshd, shell=/usr/sbin/nologin, from=none
    oct. 19 12:39:00 marceau-vm usermod[1444]: change user 'sshd' password
    oct. 19 12:39:00 marceau-vm chage[1451]: changed password expiry for sshd
    oct. 19 12:39:02 marceau-vm sshd[1567]: Server listening on 0.0.0.0 port 22.
    oct. 19 12:39:02 marceau-vm sshd[1567]: Server listening on :: port 22.
    oct. 19 12:39:25 marceau-vm sudo[2613]:  marceau : TTY=pts/0 ; PWD=/home/marceau/Desktop ; USER=root ; COMMAND=/usr/bin/systemctl enable --now sshd
  ```

---

🌞 **Connectez vous au serveur**

- depuis votre PC, en utilisant un **client SSH**

```bash
C:\Users\marce>ssh marceau@192.168.56.117
marceau@192.168.56.117's password:
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.11.0-38-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

61 updates can be applied immediately.
To see these additional updates run: apt list --upgradable

Your Hardware Enablement Stack (HWE) is supported until April 2025.
Last login: Sun Nov  7 17:00:16 2021 from 192.168.56.1
marceau@node1:~$
```

## 4. Modification de la configuration du serveur

Pour modifier comment un *service* se comporte il faut modifier le fichier de configuration. On peut tout changer à notre guise.

🌞 **Modifier le comportement du service**

- c'est dans le fichier `/etc/ssh/sshd_config`
  - c'est un simple fichier texte
  - modifiez-le comme vous voulez, je vous conseille d'utiliser `nano` en ligne de commande
- effectuez le modifications suivante :
  - changer le ***port d'écoute*** du service *SSH*
    - peu importe lequel, il doit être compris entre 1025 et 65536
  - vous me prouverez avec un `cat` que vous avez bien modifié cette ligne
  ```bash
  root@node1:~# nano /etc/ssh/sshd_config
  root@node1:~# cat /etc/ssh/sshd_config
      Port 1026
    ```
- pour cette modification, prouver à l'aide d'une commande qu'elle a bien pris effet
  - une commande `ss -l` pour vérifier le port d'écoute

> Vous devez redémarrer le service avec une commande `systemctl restart` pour que les changements inscrits dans le fichier de configuration prennent effet.

```bash
    root@node1:~# systemctl restart sshd
    root@node1:~# ss -ltpn |grep sshd
    LISTEN    0         128                0.0.0.0:1026             0.0.0.0:*        users:(("ssh",pid=27108,fd=3))
    LISTEN    0         128                   [::]:1026                [::]:*        users:(("ssh",pid=27108,fd=4))
```

🌞 **Connectez vous sur le nouveau port choisi**

- depuis votre PC, avec un *client SSH*

```bash=
C:\Users\marce>ssh -p 1026 marceau@192.168.56.117
marceau@192.168.56.117's password:
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.11.0-38-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

61 updates can be applied immediately.
To see these additional updates run: apt list --upgradable

Your Hardware Enablement Stack (HWE) is supported until April 2025.
Last login: Sun Nov  7 17:51:53 2021 from 192.168.56.1
marceau@node1:~$
```

# Partie 2 : FTP

# I. Intro

> *FTP* c'est pour *File Transfer Protocol*.

***FTP* est un protocole qui permet d'envoyer simplement des fichiers sur un serveur à travers le réseau.**

*FTP* repose un principe de client/serveur :

- le *serveur*
  - est installé sur une machine par l'admin
  - écoute sur le port 21/TCP par convention
- le *client*
  - connaît l'IP du *serveur*
  - se connecte au *serveur* à l'aide d'un programme appelé "*client FTP*"

Dans la vie réelle, *FTP* est souvent utilisé pour échanger des fichiers avec un serveur de façon sécurisée. En vrai ça commence à devenir oldschool *FTP*, mais c'est un truc très basique et toujours très utilisé.

> Si vous louez un serveur en ligne, on vous donnera parfois un accès *FTP* pour y déposer des fichiers.


# II. Setup du serveur FTP

Toujours la même routine :

- **1. installation de paquet**
  - avec le gestionnaire de paquet de l'OS
- **2. configuration** dans un fichier de configuration
  - avec un éditeur de texte
  - les fichiers de conf sont de simples fichiers texte
- **3. lancement du service**
  - avec une commande `systemctl start <NOM_SERVICE>`

> ***Pour toutes les commandes tapées qui figurent dans le rendu, je veux la commande ET son résultat. S'il manque l'un des deux, c'est useless.***

## 1. Installation du serveur

🌞 **Installer le paquet `vsftpd`**

---

Note : une fois que le paquet est installé, plusieurs nouvelles choses sont dispos sur la machine. Notamment :

- un service `vsftpd`
- un fichier de configuration `/etc/vsftpd.conf`

> Le paquet s'appelle `vsftpd` pour *Very Secure FTP Daemon*. A l'apogée de l'utilisation de FTP, il n'était pas réputé pour être un protocole très sécurisé. Aujourd'hui, avec des outils comme `vsftpd`, c'est bien mieux qu'à l'époque.
```bash=
root@node1:~# apt install vsftpd
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following NEW packages will be installed:
  vsftpd
0 upgraded, 1 newly installed, 0 to remove and 61 not upgraded.
Need to get 115 kB of archives.
After this operation, 338 kB of additional disk space will be used.
Get:1 http://fr.archive.ubuntu.com/ubuntu focal/main amd64 vsftpd amd64 3.0.3-12 [115 kB]
Fetched 115 kB in 0s (465 kB/s)
Preconfiguring packages ...
Selecting previously unselected package vsftpd.
(Reading database ... 199604 files and directories currently installed.)
Preparing to unpack .../vsftpd_3.0.3-12_amd64.deb ...
Unpacking vsftpd (3.0.3-12) ...
Setting up vsftpd (3.0.3-12) ...
Created symlink /etc/systemd/system/multi-user.target.wants/vsftpd.service → /lib/systemd/system/vsftpd.service.
Processing triggers for man-db (2.9.1-1) ...
Processing triggers for systemd (245.4-4ubuntu3.11) ...
```

## 2. Lancement du service FTP

🌞 **Lancer le service `vsftpd`**

- avec une commande `systemctl start`
- vérifier que le service est actuellement actif avec une commande `systemctl status`

> Vous pouvez aussi faire en sorte que le service FTP se lance automatiquement au démarrage avec la commande `systemctl enable vsftpd`.

```bash=
root@node1:~# systemctl start vsftpd
root@node1:~# systemctl status vsftpd
● vsftpd.service - vsftpd FTP server
     Loaded: loaded (/lib/systemd/system/vsftpd.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2021-11-07 18:36:36 CET; 1min 22s ago
   Main PID: 27445 (vsftpd)
      Tasks: 1 (limit: 2312)
     Memory: 528.0K
     CGroup: /system.slice/vsftpd.service
             └─27445 /usr/sbin/vsftpd /etc/vsftpd.conf

nov. 07 18:36:36 node1.tp2.linux systemd[1]: Starting vsftpd FTP server...
nov. 07 18:36:36 node1.tp2.linux systemd[1]: Started vsftpd FTP server.
```

## 3. Etude du service FTP

🌞 **Analyser le service en cours de fonctionnement**

- afficher le statut du service
  - avec une commande `systemctl status`
- afficher le/les processus liés au service `vsftpd`
  - avec une commande `ps`
  - isolez uniquement la/les ligne(s) intéressante(s) pour le rendu de TP
  ```bash
    root@node1:~# ps aux
    root       27445  0.0  0.1   6816  3016 ?        Ss   18:36   0:00 /usr/sbin/vsftpd /etc/vsftp
    ```

- afficher le port utilisé par le service `vsftpd`
  - avec une commande `ss -l`
  - isolez uniquement la/les ligne(s) intéressante(s)
  ```bash
  root@node1:~# ss -ltpn
  LISTEN  0       32                   *:21                 *:*      users:(("vsftpd",pid=27445,fd=3))
  ```
- afficher les logs du service `vsftpd`
  - avec une commande `journalctl`
  - en consultant un fichier dans `/var/log/`
  - ne me donnez pas toutes les lignes de logs, je veux simplement que vous appreniez à consulter les logs
  ```bash
  root@node1:~# journalctl | grep vsftpd
    nov. 07 18:36:36 node1.tp2.linux systemd[1]: Starting vsftpd FTP server...
    nov. 07 18:36:36 node1.tp2.linux systemd[1]: Started vsftpd FTP server.
    ```

---

🌞 **Connectez vous au serveur**

- depuis votre PC, en utilisant un *client FTP*
  - les navigateurs Web, ils font ça maintenant
  - demandez moi si vous êtes perdus
  ```bash=
    Hôte : 192.168.56.117
    Identifiant : marceau
    MDP : ****
    Port : 21
    ```
- essayez d'uploader et de télécharger un fichier
  - montrez moi à l'aide d'une commande la ligne de log pour l'upload, et la ligne de log pour le download
- vérifier que l'upload fonctionne
  - une fois un fichier upload, vérifiez avec un `ls` sur la machine Linux que le fichier a bien été uploadé
  ```bash
    root@node1:/home/marceau# ls
    Desktop  Documents  Downloads  Music  Pictures  Public  Templates  test.txt  Videos
    ```

🌞 **Visualiser les logs**

- mettez en évidence une ligne de log pour un download
```bash=
Mon Nov  8 15:48:55 2021 [pid 29647] [marceau] OK DOWNLOAD: Client "::ffff:192.168.56.1", "/home/marceau/test.txt", 0.00Kbyte/sec
```
- mettez en évidence une ligne de log pour un upload
```bash=
Mon Nov  8 15:43:25 2021 [pid 29603] [marceau] OK UPLOAD: Client "::ffff:192.168.56.1", "/home/marceau/test.txt", 0.00Kbyte/sec
```

## 4. Modification de la configuration du serveur

Pour modifier comment un service se comporte il faut modifier de configuration. On peut tout changer à notre guise.

🌞 **Modifier le comportement du service**

- c'est dans le fichier `/etc/vsftpd.conf`
- effectuez les modifications suivantes :
  - changer le port où écoute `vstfpd`
    - peu importe lequel, il doit être compris entre 1025 et 65536
  - vous me prouverez avec un `cat` que vous avez bien modifié cette ligne
  ```bash
    root@node1:/# cat /etc/vsftpd.conf
    Listen_port=1027
    ````
- pour les deux modifications, prouver à l'aide d'une commande qu'elles ont bien pris effet
  - une commande `ss -l` pour vérifier le port d'écoute
  ```bash
  root@node1:/# ss -ltpn
  LISTEN  0        32                     *:1027                *:*      users:(("vsftpd",pid=29683,fd=3))
  ```

> Vous devez redémarrer le service avec une commande `systemctl restart` pour que les changements inscrits dans le fichier de configuration prennent effet.

🌞 **Connectez vous sur le nouveau port choisi**

- depuis votre PC, avec un *client FTP*
```bash=
Hôte : 192.168.56.117
Identifiant : marceau
MDP : ****
Port : 1027
```
- re-tester l'upload et le download
```bash=
Mon Nov  8 16:00:54 2021 [pid 29729] [marceau] OK UPLOAD: Client "::ffff:192.168.56.1", "/home/marceau/test.txt", 0.00Kbyte/sec
Mon Nov  8 16:00:57 2021 [pid 29729] [marceau] OK DOWNLOAD: Client "::ffff:192.168.56.1", "/home/marceau/test.txt", 0.00Kbyte/sec
```

# Partie 3 : Création de votre propre service

# I. Intro

Comme on l'a dit plusieurs fois plus tôt, un *service* c'est juste un processus que l'on demande au système de lancer. Puis il s'en occupe.

Ainsi, il nous faut juste trouver comment, dans Linux, on fait pour définir un nouveau *service*. On aura plus qu'à indiquer quel processus on veut lancer.

Histoire d'avoir un truc un minimum tangible, et pas juste un service complètement inutile, on va apprendre un peu à utiliser `netcat` avant de continuer.

`netcat` est une commande très simpliste qui permet deux choses :

- **écouter sur un port** réseau, et attendre la connexion de clients
  - on parle alors d'un `netcat` qui agit comme un serveur
- **se connecter sur un port** d'un serveur dont on connaît l'IP
  - on parle alors d'un `netcat` qui agit comme un client

Dans un premier temps, vous allez utiliser `netcat` à la main et jouer un peu avec. On peut fabriquer un outil de discussion, un chat, assez facilement avec `netcat`. Un chat entre deux machines connectées sur le réseau !

Ensuite, vous créerez un service basé sur `netcat` qui permettra d'écrire dans un fichier de la machine, à distance.

> ***Pour toutes les commandes tapées qui figurent dans le rendu, je veux la commande ET son résultat. S'il manque l'un des deux, c'est useless.***

# II. Jouer avec netcat

Si vous avez compris la partie I avec SSH et la partie II avec FTP, c'est toujours le même principe : un serveur qui attend des connexions, et un client qui s'y connecte. Le principe :

- la VM va agir comme un serveur, à l'aide de la commande `netcat`
  - ce sera une commande `nc -l`
  - le `-l` c'est pour `listen` : le serveur va écouter
  - il faudra préciser un port sur lequel écouter
- votre PC agira comme le client
  - il faudra avoir la commande `nc` dans le terminal de votre PC
  - ce sera une commande `nc` (sans le `-l`) pour se connecter à un serveur
  - il faudra préciser l'IP et le port où vous voulez vous connecter (IP et port de la VM)

> Sur Windows, la commande s'appelle souvent `ncat.exe` ou `nc.exe`.


Une fois la connexion établie, vous devrez pouvoir échanger des messages entre les deux machines, comme un petit chat !

🌞 **Donnez les deux commandes pour établir ce petit chat avec `netcat`**

- la commande tapée sur la VM
```bash=
marceau@node1:~$ nc -l 1028
```
- la commande tapée sur votre PC
```bash=
marceau@marceau-vm:~$ nc 192.168.56.117 1028
```

🌞 **Utiliser `netcat` pour stocker les données échangées dans un fichier**

- utiliser le caractère `>` et/ou `>>` sur la ligne de commande `netcat` de la VM
- cela permettra de stocker les données échangées dans un fichier
- plutôt que de les afficher dans le terminal
- parce quuueeee pourquoi pas ! Ca permet de faire d'autres trucs avec
```bash=
marceau@node1:~$ nc -l 1028 > tchatNetcat.txt
marceau@node1:~$ cat tchatNetcat.txt

salut
```

Le caractère `>` s'utilise comme suit :

```bash
# On liste les fichiers et dossiers
$ ls
Documents Images

# Le caractère > permet de rediriger le texte dans un fichier
# Plutôt que de l'afficher dans un terminal
$ ls > toto
# Notez l'absence de texte en retour de la commande ls

# Voyons le résultat d'une simple commande ls désormais :
$ ls
Documents Images toto
# Un nouveau fichier toto a été créé

# Regardons son contenu
$ cat toto
Document Images
# Il contient du texte : le résultat de la commande ls
```

> N'hésitez pas à vous entraîner à utiliser `>` et `>>` sur la ligne de commande avant de vous lancer dans cette partie. Je vous laisse Google pour voir la différence entre les deux.

Il sera donc possible de l'utiliser avec `netcat` comme suit :

```bash
$ nc -l IP PORT > YOUR_FILE
```

Ainsi, le fichier `YOUR_FILE` contiendra tout ce que le client aura envoyé vers le serveur avec `netcat`, plutôt que ça s'affiche juste dans le terminal.

> Renommez le fichier `YOUR_FILE` comme vous l'entendez évidemment.

# III. Un service basé sur netcat

**Pour créer un service sous Linux, il suffit de créer un simple fichier texte.**

Ce fichier texte :

- a une syntaxe particulière
- doit se trouver dans un dossier spécifique

Pour essayer de voir un peu la syntaxe, vous pouvez utilisez la commande `systemctl cat` sur un service existant. Par exemple `systemctl cat sshd`.

DON'T PANIC pour votre premier service j'vais vous tenir la main.

La commande que lancera votre service sera un `nc -l` : vous allez donc créer un petit chat sous forme de service ! Ou presque hehe.

## 1. Créer le service

🌞 **Créer un nouveau service**

- créer le fichier `/etc/systemd/system/chat_tp2.service`
- définissez des permissions identiques à celles des aux autres fichiers du même type qui l'entourent
- déposez-y le contenu suivant :

```bash
[Unit]
Description=Little chat service (TP2)

[Service]
ExecStart=<NETCAT_COMMAND>

[Install]
WantedBy=multi-user.target
```

Vous devrez remplacer `<NETCAT_COMMAND>` par une commande `nc` de votre choix :

- `nc` doit écouter, listen (`-l`)
- et vous devez préciser le chemin absolu vers cette commande `nc`
  - vous pouvez taper la commande `which nc` pour connaître le dossier où se trouve `nc` (son chemin absolu)

```bash=
marceau@node1:~$ sudo nano /etc/systemd/system/chat_tp2.service
[sudo] password for marceau:

marceau@node1:~$ cat /etc/systemd/system/chat_tp2.service
[Unit]
Description=Little chat service (TP2)

[Service]
ExecStart=nc -l 1028

[Install]
WantedBy=multi-user.target

marceau@node1:~$ sudo chmod 777 /etc/systemd/system/chat_tp2.service
```
**Il faudra exécuter la commande `sudo systemctl daemon-reload` à chaque fois que vous modifiez un fichier `.service`.**

## 2. Test test et retest

🌞 **Tester le nouveau service**

- depuis la VM
  - démarrer le nouveau service avec une commande `systemctl start`
  ```bash
  marceau@node1:~$ systemctl start chat_tp2.service
    ==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ===
    Authentication is required to start 'chat_tp2.service'.
    Authenticating as: marceau,,, (marceau)
    Password:
    ==== AUTHENTICATION COMPLETE ===
    ```
  - vérifier qu'il est correctement lancé avec une commande  `systemctl status`
     ```bash
     marceau@node1:~$ systemctl status chat_tp2.service
    ● chat_tp2.service - Little chat service (TP2)
     Loaded: loaded (/etc/systemd/system/chat_tp2.service; disabled; vendor    preset: enable>     Active: active (running) since Mon 2021-11-08 17:20:37 CET; 16s ago
       Main PID: 30425 (nc)
      Tasks: 1 (limit: 2312)
     Memory: 184.0K
     CGroup: /system.slice/chat_tp2.service
             └─30425 /usr/bin/nc -l 1028

    nov. 08 17:20:37 node1.tp2.linux systemd[1]: Started Little chat service (TP2).
    ```
  - vérifier avec une comande `ss -l` qu'il écoute bien derrière le port que vous avez choisi
  ```bash
  marceau@node1:~$ ss -ltpn
  LISTEN    0         1                  0.0.0.0:1028             0.0.0.0:*
  ```
- tester depuis votre PC que vous pouvez vous y connecter
```bash=
marceau@marceau-vm:~$ nc 192.168.56.117 1028

oui
```
- pour visualiser les messages envoyés par le client, il va falloir regarder les logs de votre service, sur la VM :

```bash
# Voir l'état du service, et les derniers logs
$ systemctl status chat_tp2

# Voir tous les logs du service
$ journalctl -xe -u chat_tp2

# Suivre en temps réel l'arrivée de nouveaux logs
# -f comme follow :)
$ journalctl -xe -u chat_tp2 -f
```
```bash=
marceau@node1:~$ journalctl -xe -u chat_tp2 -f
-- Logs begin at Tue 2021-10-19 11:34:27 CEST. --
nov. 08 17:20:37 node1.tp2.linux systemd[1]: Started Little chat service (TP2).
-- Subject: A start job for unit chat_tp2.service has finished successfully
-- Defined-By: systemd
-- Support: http://www.ubuntu.com/support
--
-- A start job for unit chat_tp2.service has finished successfully.
--
-- The job identifier is 13854.
nov. 08 17:27:47 node1.tp2.linux nc[30425]: oui
nov. 08 17:27:50 node1.tp2.linux systemd[1]: chat_tp2.service: Succeeded.
-- Subject: Unit succeeded
-- Defined-By: systemd
-- Support: http://www.ubuntu.com/support
--
-- The unit chat_tp2.service has successfully entered the 'dead' state.
```
