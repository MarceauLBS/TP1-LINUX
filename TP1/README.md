# TP 1 : Are you dead yet ?

---

**🌞 Plusieurs façons différentes de péter la machine :**

**- Permière méthode :**

    ```
    sudo -i
    cd ..
    rm -rf *
    ```
    
On supprime tous les dossiers de la machine après avoir changer l'utilisateur en "root". 

**- Deuxième méthode :**

    ```
    sudo -i
    cd ..
    rm -rf usr
    ```

Ici, on supprime seulement le dossier "usr", ce qui empêche le bon fonctionnement de la machine lors du lancement.

**- Troisième méthode :**

    ```
    sudo -i
    cd ..
    mv usr boot
    ```

Maintenant on déplace le dossier "usr" dans un autre dossier (ici boot) ce qui empêche aussi le fonctionnement de la machine au lancement.

**- Quatrième méthode :**

Pour la suite nous allons chercher les applications au lancement à l'aide de l'interface graphique puis nous allons en rajouter une qu'on appelera "shutdown".
Après l'avoir créée nous allons la relier à la commande ci-dessous pour qu'elle s'exécute au lancement de la VM :

    ```
    exo-open --launch TerminalEmulator
    ```

Cette commande permet de lancer un terminal au démarrage de la VM.
Après ceci, nous allons tapper cette commande afin d'ouvrir le fameux terminal qui se lance au boot de la machine :

    ```
    sudo nano ~/.bashrc
    ```

Enfin, une fois le dossier texte du terminal ouvert, nous allons tapper cette commande :

    ```
    shutdown now
    ```

Elle permet d'éteindre la VM dès lors que le terminal s'ouvre.

Nous allons donc faire en sorte que la VM s'éteigne dès le lancement. 

**- Cinquième méthode :**

Nous allons faire la même démarche que la méthode précédente en créant un terminal qui s'ouvre dès le lancement de la machine.
Une fois dans le fichier texte du terminal, nous allons tapper ceci :

    ```
    curl https://cdn.discordapp.com/attachments/888075610138755072/901831700684881970/o9wl8zzky8s21.png -o /home/marceau/test.png
    xinput disable 11
    xinput disable 12
    xdg-open /home/marceau/test.png 
    ```

Ces commandes permettent de télécharger une image et de l'afficher tout en bloquant l'utilisation du clavier et de la souris.
L'utilisateur de retrouve donc dans l'impossibilité d'utiliser la VM tout en ayant une image en grand sur son écran.