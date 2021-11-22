# TP 3 : A little script

- [TP 3 : A little script](#tp-3--a-little-script)
- [Intro](#intro)
- [I. Script carte d'identité](#i-script-carte-didentité)
  - [Rendu](#rendu)
- [II. Script youtube-dl](#ii-script-youtube-dl)
  - [Rendu](#rendu-1)
- [III. MAKE IT A SERVICE](#iii-make-it-a-service)
  - [Rendu](#rendu-2)
- [Bonus](#iv-bonus)

# Intro

Aujourd'hui un TP pour apprendre un peu **le scripting**.

Le scripting dans GNU/Linux, c'est simplement le fait d'écrire dans un fichier une suite de commande, qui seront exécutées les unes à la suite des autres lorsque l'on exécutera le script.

Plus précisément, on utilisera la syntaxe du shell `bash`. Et on a le droit à l'algo (des conditions `if`, des boucles `while`, etc).

Bon par contre, la syntaxe `bash`, elle fait mal aux dents. Ca va prendre un peu de temps pour s'habituer. Pour ça, vous **devez** prendre connaissance des deux ressources suivantes :

➜ Pour écrire des scripts, on manipule beaucoup des flux des commandes, allez check [le cours-notion sur le sujet si c'est pas clair pour vous](../../cours/notions/flux/README.md).

➜ Pour la syntaxe de l'algo en `bash` je vous renvoie vers ce site : https://devhints.io/bash.

# I. Script carte d'identité

Vous allez écrire **un script qui récolte des informations sur le système et les affiche à l'utilisateur.** Il s'appellera `idcard.sh` et sera stocké dans `/srv/idcard/idcard.sh`.

> `.sh` est l'extension qu'on donne par convention aux scripts réalisés pour être exécutés avec `sh` ou `bash`.

Dans un premier temps, essayez de trouver le commandes permettant de réaliser telle ou telle action.

Ne commencez la rédaction du script que dans un second temps, une fois que vous savez quelles commandes vont pouvoir vous servir pour récupérer telle ou telle info.

Ce que doit faire le script. Il doit afficher :

- le nom de la machine
- le nom de l'OS de la machine
- la version du noyau Linux utilisé par la machine
- l'adresse IP de la machine
- l'état de la RAM
  - espace dispo en RAM (en Go, Mo, ou Ko)
  - taille totale de la RAM (en Go, Mo, ou ko)
- l'espace restant sur le disque dur, en Go (ou Mo, ou ko)
- le top 5 des processus qui pompent le plus de RAM sur la machine actuellement. Procédez par étape :
  - listez les process
  - affichez la RAM utilisée par chaque process
  - triez par RAM utilisée
  - isolez les 5 premiers
- la liste des ports en écoute sur la machine, avec le programme qui est derrière
- un lien vers une image/gif random de chat
  - il y a de très bons sites pour ça hihi
  - avec [celui-ci](https://docs.thecatapi.com/), une simple commande `curl https://api.thecatapi.com/v1/images/search` vous retourne l'URL d'une random image de chat

Pour vous faire manipuler les sorties/entrées de commandes, votre script devra sortir **exactement** :

```bash
$ /srv/idcard/idcard.sh
Machine name : ...
OS ... and kernel version is ...
IP : ...
RAM : ... RAM restante sur ... RAM totale
Disque : ... space left
Top 5 processes by RAM usage :
  - ...
  - ...
  - ...
  - ...
  - ...
Listening ports :
  - 22 : sshd
  - ...
  - ...

Here's your random cat : https://cdn2.thecatapi.com/images/ahb.jpg
```

## Rendu

📁 **Fichier `/srv/idcard/idcard.sh`**

🌞 Vous fournirez dans le compte-rendu, en plus du fichier, **un exemple d'exécution avec une sortie**, dans des balises de code.

```bash=
marceau@marceau-vm:/srv/idcard$ cat idcard.sh
hostname=$(hostname)
echo "Machine name : $hostname"
osname=$(head -n 1 /etc/os-release | cut -d'"' -f2)
kernel=$(uname -ar | cut -d' ' -f3)
echo "OS $osname and kernel version is $kernel"
ip=$(hostname -I | cut -d' ' -f2)
echo "IP : $ip"
ramT=$(free -mh | grep Mem | tr -s ' ' | cut -d' ' -f2)
ramL=$(free -mh | grep Mem | tr -s ' ' | cut -d' ' -f4)
echo "RAM : $ramL Mi RAM restante sur $ramT Mi RAM totale"
disqueD=$(df -t ext4 -h | tail -n 1 | tr -s ' ' | cut -d' ' -f4 | rev | cut -c2- | rev)
echo "Disque : $disqueD Gi space left"
ps -o %mem,command ax | sort -b -k1 -r | head -n 6 | tail -n 5 > /srv/idcard/.process
echo "Top 5 processes by RAM usage :"
cat /srv/idcard/.process | while read line; do
        echo $line > /srv/idcard/.process2
        echo "- $(cat /srv/idcard/.process2 |cut -d' ' -f2)"
done
echo $(curl -s https://api.thecatapi.com/v1/images/search | cut -d'"'  -f10)

marceau@marceau-vm:/srv/idcard$ ./idcard.sh
Machine name : marceau-vm
OS Ubuntu and kernel version is 5.11.0-38-generic
IP : 192.168.56.120
RAM : 227Mi Mi RAM restante sur 1,9Gi Mi RAM totale
Disque : 1,8 Gi space left
Top 5 processes by RAM usage :
- /usr/bin/python3
- /usr/bin/python3
- xfwm4
- /usr/lib/xorg/Xorg
- /usr/libexec/fwupd/fwupd
https://cdn2.thecatapi.com/images/v8n1OkKUi.jpg
```

# II. Script youtube-dl

**Un petit script qui télécharge des vidéos Youtube.** Vous l'appellerez `yt.sh`. Il sera stocké dans `/srv/yt/yt.sh`.

**Pour ça on va avoir besoin d'une commande : `youtube-dl`.** Je vous laisse vous référer [à la doc officielle](https://github.com/ytdl-org/youtube-dl/blob/master/README.md#readme) pour voir comment récupérer cette commande sur votre machine.

Comme toujours, **PRENEZ LE TEMPS** de manipuler la commande et d'explorer un peu le `youtube-dl --help`.

Le contenu de votre script :

➜ **1. Permettre le téléchargement d'une vidéo youtube dont l'URL est passée au script**

- la vidéo devra être téléchargée dans le dossier `/srv/yt/downloads/`
  - le script doit s'assurer que ce dossier existe sinon il quitte
  - vous pouvez utiliser la commande `exit` pour que le script s'arrête
- plus précisément, chaque téléchargement de vidéo créera un dossier
  - `/srv/yt/downloads/<NOM_VIDEO>`
  - il vous faudra donc, avant de télécharger la vidéo, exécuter une commande pour récupérer son nom afin de créer le dossier en fonction
- la vidéo sera téléchargée dans
  - `/srv/yt/downloads/<NOM_VIDEO>/<NOM_VIDEO>.mp4`
- la description de la vidéo sera aussi téléchargée
  - dans `/srv/yt/downloads/<NOM_VIDEO>/description`
  - on peut récup la description avec une commande `youtube-dl`
- la commande `youtube-dl` génère du texte dans le terminal, ce texte devra être masqué
  - vous pouvez utiliser une redirection de flux vers `/dev/null`, c'est ce que l'on fait généralement pour se débarasser d'une sortie non-désirée

Il est possible de récupérer les arguments passés au script dans les variables `$1`, `$2`, etc.

```bash
$ cat script.sh
echo $1

$ ./script.sh toto
toto
```

➜ **2. Le script produira une sortie personnalisée**

- utilisez la commande `echo` pour écrire dans le terminal
- la sortie devra être comme suit :

```bash
$ /srv/yt/yt.sh https://www.youtube.com/watch?v=sNx57atloH8
Video https://www.youtube.com/watch?v=sNx57atloH8 was downloaded. 
File path : /srv/yt/downloads/tomato anxiety/tomato anxiety.mp4`
```

➜ **3. A chaque vidéo téléchargée, votre script produira une ligne de log dans le fichier `/var/log/yt/download.log`**

- votre script doit s'assurer que le dossier `/var/log/yt/` existe, sinon il refuse de s'exécuter
- la ligne doit être comme suit :

```
[yy/mm/dd hh:mm:ss] Video <URL> was downloaded. File path : <PATH>`
```

Par exemple :

```
[21/11/12 13:22:47] Video https://www.youtube.com/watch?v=sNx57atloH8 was downloaded. File path : /srv/yt/downloads/tomato anxiety/tomato anxiety.mp4`
```

Hint : La commande `date` permet d'afficher la date et de choisir à quel format elle sera affichée. Idéal pour générer des logs. [J'ai trouvé ce lien](https://www.geeksforgeeks.org/date-command-linux-examples/), premier résultat google pour moi, y'a de bons exemples (en bas de page surtout pour le formatage de la date en sortie).

## Rendu

📁 **Le script `/srv/yt/yt.sh`**

📁 **Le fichier de log `/var/log/yt/download.log`**, avec au moins quelques lignes

🌞 Vous fournirez dans le compte-rendu, en plus du fichier, **un exemple d'exécution avec une sortie**, dans des balises de code.

```bash=
marceau@marceau-vm:/srv/yt$ cat yt.sh
#!/bin/bash
usage() {
        echo "Usage: yt.sh [options] <url>"
        echo " -q <quality> "
        echo " -o <directory> path file"
        echo " -h to show this menu "
        exit
}
if [[ -d "/srv/yt/downloads" && -d "/var/log/yt/" ]]; then
        path="/srv/yt/downloads"
        form="mp4"
        for i in "$@"; do
                if [[ "$i" == "-q" ]]; then
                        path=true
                elif [[ "$i" == "-o" ]]; then
                        form=true
                elif [[ "$i" == "-h" ]]; then
                        echo "Usage : yt.sh [options] url"
                        echo " Options : -q <quality>   -o <directory> File path"
                        exit
                elif [[ $form == true ]]; then
                        form="$i"
                elif [[ $path == true ]]; then
                        path="$i"
                fi
                link="$i"
        done
        if [[ "$link" =~ https://www.youtube.com/ ]]; then
                if youtube-dl -e "$link" &> /dev/null; then
                        titre=$(youtube-dl -e "$link")
                        mkdir "$path/$titre" &> /dev/null
                        if (cd "$path/$titre" && youtube-dl -f "$form/mp4" "$link" &>/dev/null); then
                                youtube-dl --get-description "$link" > "$path/$titre/desc"
                                echo "Video $link was downloaded in $form form."
                                echo "File path : $path/$titre/$titre.mp4"
                                echo "[$(date "+%D %T")]"" Video $link was downloaded. File path : ""$path/$titre/$titre.mp4""'" >> "/var/log/yt/download.log"
                        else
                                echo "Video download failed"
                echo "[$(date "+%D %T")]"" Video $link failed to download.""'" >> "/var/log/yt/download.log"
                                usage
                        fi
                else
            echo "[$(date "+%D %T")]"" Link $link is not working.""'" >> "/var/log/yt/download.log"
                        echo "Link not working"
                        usage
                fi
        else
        echo "[$(date "+%D %T")]"" Wrong URL : $link""'" >> "/var/log/yt/download.log"
                echo "wrong URL"
                usage
        fi
else
        echo "Missing file"
fi

marceau@marceau-vm:/srv/yt$ cd downloads/
marceau@marceau-vm:/srv/yt/downloads$ ls
'c'\''est pas possible, c'\''est pas possible......(video drole enfant )'
marceau@marceau-vm:/srv/yt/downloads$ cd c\'est\ pas\ possible\,\ c\'est\ pas\ possible......\(video\ drole\ enfant\ \)/
marceau@marceau-vm:/srv/yt/downloads/c'est pas possible, c'est pas possible......(video drole enfant )$ ls
'c'\''est pas possible, c'\''est pas possible......(video drole enfant )-laYD6hZ6tcw.mp4'   description
marceau@marceau-vm:/srv/yt/downloads/c'est pas possible, c'est pas possible......(video drole enfant )$ cat description
abonne toi
```

Le fichier de log /var/log/yt/download.log :

```
[11/19/22 21:36:46]Video https://www.youtube.com/watch?v=laYD6hZ6tcw was downloaded. File path : /srv/yt/downloads/c'est pas possible/c'est pas possible.mp4'
[11/19/22 21:36:43]dgqsdgqdjqgfq is not working'
```

# III. MAKE IT A SERVICE

YES. Yet again. **On va en faire un [service](../../cours/notions/serveur/README.md#ii-service).**

L'idée :

➜ plutôt que d'appeler la commande à la main quand on veut télécharger une vidéo, **on va créer un service qui les téléchargera pour nous**

➜ le service devra **lire en permanence dans un fichier**

- s'il trouve une nouvelle ligne dans le fichier, il vérifie que c'est bien une URL de vidéo youtube
  - si oui, il la télécharge, puis enlève la ligne
  - sinon, il enlève juste la ligne

➜ qui écrit dans le fichier pour ajouter des URLs ? Bah vous !

- vous pouvez écrire une liste d'URL, une par ligne, et le service devra les télécharger une par une

---

Pour ça, procédez par étape :

- partez de votre script précédent (gardez une copie propre du premier script)
  - le nouveau script s'appellera `yt-v2.sh`
- adaptez le pour qu'il lise les URL dans un fichier plutôt qu'en argument
- faites en sorte qu'il tourne en permanence, et vérifie le contenu du fichier toutes les X secondes
- il doit marcher si on précise une vidéo par ligne
  - il les télécharge une par une
  - et supprime les lignes une par une
- enfin, créez un service qui le lance :
  - créez un fichier `/etc/systemd/system/yt.service`

```bash
[Unit]
Description=<Votre description>

[Service]
ExecStart=<Votre script>

[Install]
WantedBy=multi-user.target
```

> Pour rappel, après la moindre modification dans le dossier `/etc/systemd/system/`, vous devez exécuter la commande `sudo systemctl daemon-reload` pour dire au système de lire les changements qu'on a effectué.

Vous pourrez alors interagir avec votre service à l'aide des commandes habituelles `systemctl` :

- `systemctl status yt`
- `sudo systemctl start yt`
- `sudo systemctl stop yt`

> Vérifiez aussi que votre script génère toujours des logs à chaque téléchargement !

## Rendu

📁 **Le script `/srv/yt/yt-v2.sh`**

📁 **Fichier `/etc/systemd/system/yt.service`**

🌞 Vous fournirez dans le compte-rendu, en plus des fichiers :

Le script /srv/yt/yt-v2.sh :
```bash=
#!/bin/bash
while true; do
    if [[ -d "/srv/yt/downloads" && -d "/var/log/yt/" && -s "/srv/yt/url" ]]; then 
        while read -r line < /srv/yt/url; do
            if [[ "$line" =~ https://www.youtube.com/ ]]; then
                if youtube-dl -e "$line" &> /dev/null; then
                    titre=$(youtube-dl -e "$line") 
                    mkdir "/srv/yt/downloads/$titre"
                    youtube-dl -o "/srv/yt/downloads/$titre/$titre.mp4" --format mp4 "$line" > /dev/null
                    youtube-dl --get-description "$line" > "/srv/yt/downloads/$titre/description"
                    echo "Video $line was downloaded"
                    echo "File path : /srv/yt/downloads/$titre/$titre.mp4"
                    echo "[$(date "+%D %T")]"" Video $line was downloaded. File path : ""/srv/yt/downloads/$titre/$titre.mp4""'" >> "/var/log/yt/download.log"
                    sed -i '1d' /srv/yt/url
                else
                    echo "[$(date "+%D %T")]""$line is not working""'" >> "/var/log/yt/download.log"
                    sed -i '1d' /srv/yt/url
                fi
            else 
                echo "[$(date "+%D %T")]"" Wrong URL : $line""'" >> "/var/log/yt/download.log"
                echo "wrong URL"
                sed -i '1d' /srv/yt/url
            fi
        done
    fi
done
```
Fichier /etc/systemd/system/yt.service :

```
[Unit]
Description="Installation de vidéo yt"

[Service]
ExecStart=/usr/bin/bash /srv/yt/yt-v2.sh

[Install]
WantedBy=multi-user.target
```


- un `systemctl status yt` quand le service est en cours de fonctionnement
- un extrait de `journalctl -xe -u yt`
- la commande qui permet à ce service de démarrer automatiquement quand la machine démarre

> Hé oui les commandes `journalctl` fonctionnent sur votre service pour voir les logs ! Et vous devriez constater que c'est vos `echo` qui pop. En résumé, **le STDOUT de votre script, c'est devenu les logs du service !**

🌟**BONUS** : get fancy. Livrez moi un gif ou un [asciinema](https://asciinema.org/) (PS : c'est le feu asciinema) de votre service en action, où on voit les URLs de vidéos disparaître, et les fichiers apparaître dans le fichier de destination

# IV. Bonus

Quelques bonus pour améliorer le fonctionnement de votre script :

➜  en accord avec les règles de [ShellCheck](https://www.shellcheck.net/)

- bonnes pratiques, sécurité, lisibilité

➜  fonction `usage`

- le script comporte une fonction `usage`
- c'est la fonction qui est appelée lorsque l'on appelle le script avec une erreur de syntaxe
- ou lorsqu'on appelle le `-h` du script
    ²
➜  votre script a une gestion d'options :

- `-q` pour préciser la qualité des vidéos téléchargées (on peut choisir avec `youtube-dl`)
- `-o` pour préciser un dossier autre que `/srv/yt/`
- `-h` affiche l'usage

➜  si votre script utilise des commandes non-présentes à l'installation (`youtube-dl`, `jq` éventuellement, etc.)

- vous devez TESTER leur présence et refuser l'exécution du script

➜  si votre script a besoin de l'existence d'un dossier ou d'un utilisateur

- vous devez tester leur présence, sinon refuser l'exécution du script

➜ pour le téléchargelent des vidéos

- vérifiez à l'aide d'une expression régulière que les strings ssaisies dans le fichier sont bien des URLs de vidéos Youtube