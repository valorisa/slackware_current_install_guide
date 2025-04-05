# Guide d'Installation de Slackware Current

Ce document décrit les étapes pour installer Slackware "Current", la branche de développement de Slackware Linux. Ce guide est basé sur les informations et procédures discutées en date du **Samedi 5 avril 2025**.

**ATTENTION :** Slackware Current est une version de **développement/test**. Elle reçoit des mises à jour fréquentes, peut contenir des bugs et n'est généralement pas recommandée pour les débutants ou les environnements de production nécessitant une stabilité maximale. Utilisez-la à vos propres risques.

## Prérequis

1. **Sauvegarde :** Sauvegardez toutes vos données importantes avant de commencer. Le partitionnement effacera des données.
2. **Image ISO :** Téléchargez l'image ISO d'installation de Slackware Current (64 bits recommandée) depuis un miroir officiel.
    * Exemple de répertoire miroir : `https://mirrors.slackware.com/slackware/slackware-current/`
    * Cherchez un fichier comme `slackware64-current-install-dvd.iso`.
    * (Optionnel mais recommandé) Vérifiez la somme de contrôle (MD5/SHA) de l'ISO téléchargée.
3. **Clé USB :** Une clé USB d'au moins 4 Go (8 Go+ recommandé). **Son contenu sera effacé.**
4. **Configuration BIOS/UEFI :**
    * Configurez l'ordre de démarrage pour démarrer depuis la clé USB.
    * Désactivez le `Secure Boot`.
    * Si vous installez en parallèle avec Windows, désactivez le `Démarrage rapide` (Fast Startup) dans les options d'alimentation de Windows.

## Création de la Clé USB Amorçable

**AVERTISSEMENT : L'étape suivante effacera TOUT le contenu de la clé USB spécifiée (`/dev/sdX` ou `/dev/rdiskX`). Soyez absolument certain d'utiliser le bon nom de périphérique !**

### Sous Linux

1. Identifiez votre clé USB (ex: `/dev/sdb`, `/dev/sdc`) avec `lsblk` ou `sudo fdisk -l`. **Ne vous trompez pas de disque !**
2. Assurez-vous que les partitions de la clé ne sont pas montées (`sudo umount /dev/sdX*`).
3. Utilisez `dd` :

    ```bash
    sudo dd if=/chemin/vers/slackware64-current-install-dvd.iso of=/dev/sdX bs=1M status=progress oflag=sync
    ```

    (Remplacez `/chemin/vers/...iso` et `/dev/sdX` par vos valeurs).

### Sous Windows

Utilisez un outil graphique :

* **Rufus** ([https://rufus.ie/](https://rufus.ie/)):
  * Sélectionnez le périphérique USB.
  * Sélectionnez l'image ISO.
  * Si demandé, choisissez le mode **"Image DD"** pour l'écriture.
  * Cliquez sur DÉMARRER et confirmez l'effacement.
* **balenaEtcher** ([https://www.balena.io/etcher/](https://www.balena.io/etcher/)):
  * Flash from file -> Sélectionnez l'ISO.
  * Select target -> Sélectionnez la clé USB.
  * Flash! -> Confirmez.

### Sous macOS

1. Identifiez votre clé USB (ex: `/dev/disk2`, `/dev/disk3`) avec `diskutil list`.
2. Démontez la clé : `diskutil unmountDisk /dev/diskX`.
3. Utilisez `dd` (avec `rdisk` pour la vitesse) :

    ```bash
    sudo dd if=/chemin/vers/slackware64-current-install-dvd.iso of=/dev/rdiskX bs=1m
    ```

    (Remplacez `/chemin/vers/...iso` et `/dev/rdiskX` par vos valeurs).
4. Pour suivre la progression :
    * Appuyez sur `Ctrl+T` pendant l'exécution pour un statut ponctuel.
    * Ou installez `pv` (`brew install pv`) et utilisez :

        ```bash
        sudo sh -c "pv < /chemin/vers/slackware...iso | dd of=/dev/rdiskX bs=1m"
        ```

## Partitionnement du Disque Dur

1. Démarrez l'ordinateur depuis la clé USB créée. Appuyez sur `Entrée` à l'écran de démarrage.
2. (Optionnel) Configurez la disposition du clavier (ex: `fr-latin9.map`) si proposé, ou faites-le plus tard dans `setup`.
3. Connectez-vous en tant que `root` (pas de mot de passe requis).
4. Identifiez le disque dur cible (ex: `/dev/sda`, `/dev/nvme0n1`) avec `fdisk -l`.
5. Lancez un outil de partitionnement. **`cfdisk` est recommandé pour sa relative simplicité :**

    ```bash
    cfdisk /dev/sdX
    ```

    (Remplacez `/dev/sdX` par votre disque dur cible).
6. **Créez au minimum :**
    * Une partition **Swap** : Type `Linux swap` (code 82). Taille : 1x RAM ou plus (si hibernation), sinon 1-4 Go suffisent souvent.
    * Une partition **Racine (`/`)** : Type `Linux` (code 83). Taille : 20 Go minimum, 50 Go+ recommandé pour une installation complète. Utilisez l'espace restant ou la taille désirée.
    * (Optionnel) Une partition `/home` séparée (Type `Linux`).
    * Si vous utilisez UEFI, vous aurez besoin d'une partition **EFI System Partition (ESP)** existante ou à créer (Type `EFI System`, ~500 Mo, formatée en FAT32). `cfdisk` et `setup` la gèrent bien.
7. Dans `cfdisk`, utilisez les flèches, `New`, `Type`, `Write` (pour écrire les changements - **Attention, destructif !**), et `Quit`.

## Installation via `setup`

1. Lancez le programme d'installation :

    ```bash
    setup
    ```

2. Naviguez dans les menus avec les flèches et validez avec `Entrée`. Suivez ces étapes :
    * **`KEYMAP` :** Configurez votre disposition de clavier (ex: `fr-latin9.map`) si ce n'est pas déjà fait.
    * **`ADDSWAP` :** Sélectionnez votre partition swap. Confirmez sa configuration et son activation. Laissez l'installateur vérifier les blocs défectueux (rapide).
    * **`TARGET` :** Sélectionnez votre partition racine (`/`). Choisissez de la **formater** avec le système de fichiers `ext4` (choix sûr et courant). Si vous avez une partition `/home` séparée, vous la sélectionnerez et la formaterez aussi ici. La partition EFI (si UEFI) sera aussi détectée ici pour être montée sur `/boot/efi` (ne pas formater si elle contient déjà un autre OS).
    * **`SOURCE` :** Choisissez `1` (Install from Slackware USB stick) ou `2` (CD/DVD). Laissez l'installateur scanner (`Auto` ou `Scan`) pour trouver le répertoire `slackware64`.
    * **`SELECT` :** Choisissez les séries de paquets à installer. Pour une installation de bureau complète, laissez toutes les séries recommandées (`[*]`) sélectionnées. C'est le choix le plus simple.
    * **`INSTALL` :** Choisissez le mode d'installation. `full` est fortement recommandé pour installer tous les paquets des séries sélectionnées, évitant des problèmes de dépendances manuelles plus tard. Patientez pendant l'installation.
    * **`CONFIGURE` :** Étape cruciale de configuration :
        * *Make USB Boot Stick :* Optionnel (`Skip`).
        * *Install LILO/ELILO :* **Essentiel !** Choisissez `simple` pour une installation automatique sur le MBR (BIOS) ou l'ESP (UEFI). Vérifiez bien la cible. LILO pour BIOS, ELILO pour UEFI.
        * *Mouse Configuration :* Configurez `gpm` pour la souris en console (ex: `imps2`).
        * *Network Configuration :* Configurez le réseau.
            * `Hostname` (ex: `slackcurrent`).
            * `Domain name` (ex: `localdomain`).
            * Méthode IP : `NetworkManager` (recommandé pour bureau/portable), `DHCP` (si géré par routeur/box), ou `Static`.
            * Confirmez les paramètres.
        * *Startup Services :* Choisissez les services à lancer au démarrage. Les défauts sont souvent corrects. Activez `NetworkManager` si vous l'avez choisi. `sshd` pour accès distant.
        * *Custom Screen Fonts :* Optionnel (`Skip`).
        * *Hardware Clock Set To :* Choisissez `UTC` si Slackware est le seul OS, ou `localtime` si vous êtes en dual-boot avec Windows.
        * *Timezone Configuration :* Sélectionnez votre fuseau horaire (ex: `Europe/Paris`).
        * *Default Window Manager :* Choisissez l'environnement de bureau par défaut si plusieurs sont installés (ex: `XFCE`, `Plasma`).
        * *Set Root Password :* **Définissez un mot de passe `root` solide et ne l'oubliez pas !**
    * **`EXIT` :** Quittez l'installateur `setup`.

## Post-Installation

1. Retirez la clé USB.
2. Redémarrez l'ordinateur :

    ```bash
    reboot
    ```

3. Connectez-vous en tant que `root` avec le mot de passe défini.
4. **Créez un utilisateur standard** (fortement recommandé) :

    ```bash
    adduser votre_nom_utilisateur
    ```

    Suivez les invites (mot de passe, etc.). Ajoutez cet utilisateur aux groupes utiles si besoin (ex: `wheel`, `audio`, `video`, `cdrom`, `plugdev`, etc.) avec `usermod -a -G group1,group2,... votre_nom_utilisateur`.
5. Déconnectez-vous (`exit` ou `logout`) et reconnectez-vous avec votre nouvel utilisateur.
6. Démarrez l'interface graphique (si installée) :

    ```bash
    startx
    ```

7. Configurez le Wi-Fi si besoin (si vous utilisez NetworkManager) :
    * En ligne de commande : `nmtui`
    * Graphiquement : Utilisez l'applet NetworkManager dans votre barre des tâches.

## Maintenance de Slackware Current

* **Nature de "Current" :** Rappel : "Current" est une branche de test. Attendez-vous à des changements fréquents et potentiellement des instabilités. N'utilisez cette branche que si vous êtes prêt à potentiellement dépanner votre système.
* **Mises à jour :** Il est crucial de mettre à jour régulièrement pour bénéficier des derniers correctifs et fonctionnalités, mais aussi pour éviter un trop grand écart avec la branche qui pourrait rendre les mises à jour futures difficiles. La procédure standard utilise l'outil `slackpkg`.

    1. **Configurer le Miroir (`/etc/slackpkg/mirrors`) :** C'est la **première étape indispensable** avant de pouvoir utiliser `slackpkg`. Vous devez lui indiquer où télécharger les informations et les paquets.
        * Ouvrez le fichier de configuration des miroirs avec un éditeur de texte en tant que `root` :

            ```bash
            sudo nano /etc/slackpkg/mirrors
            # ou utilisez ee, vi, emacs...
            ```

        * Ce fichier contient une longue liste de miroirs du monde entier, tous commentés par défaut (lignes commençant par `#`).
        * **Règle d'or :** Vous devez décommenter (supprimer le `#` au début) **une seule ligne** dans tout le fichier. Cette unique ligne active indiquera à `slackpkg` quel serveur utiliser.
        * **Choix du miroir :** Pour de meilleures performances depuis votre localisation (Montpellier), il est fortement recommandé de choisir un miroir physiquement proche (en France ou Europe de l'Ouest) qui héberge la branche que vous utilisez (`slackware64-current` dans ce cas). Le protocole HTTPS est généralement préférable à HTTP ou FTP.
        * **Exemple (choisir le miroir IRCAM en France) :** Faites défiler le fichier jusqu'à trouver la section des miroirs français. Repérez un miroir comme celui de l'IRCAM et décommentez sa ligne :

            ```text
            # ... (beaucoup d'autres miroirs commentés avant et après) ...

            # ==========================================================================
            # = Miroirs situés en France                                                =
            # ==========================================================================

            # Institut de Recherche et Coordination Acoustique/Musique (IRCAM), Paris (HTTPS)
            # Rapide, fiable et en France. Un excellent choix depuis Montpellier.
            # Pour l'utiliser, supprimez le '#' au début de la ligne suivante :
            # (LIGNE ORIGINALEMENT COMMENTÉE) :
            # [https://mirrors.ircam.fr/pub/slackware/slackware64-current/](https://mirrors.ircam.fr/pub/slackware/slackware64-current/)
            # (LIGNE À DÉCOMMENTER - SANS LE # DEVANT) :
            [https://mirrors.ircam.fr/pub/slackware/slackware64-current/](https://mirrors.ircam.fr/pub/slackware/slackware64-current/)

            # ... (assurez-vous que TOUTES les autres lignes de miroirs restent commentées !) ...
            ```

        * Sauvegardez les modifications (ex: `Ctrl+O` avec nano) et quittez l'éditeur (ex: `Ctrl+X` avec nano).

    2. **Tester le miroir et mettre à jour les listes de paquets :** Une fois le miroir configuré, lancez (en `root` ou avec `sudo`):

        ```bash
        sudo slackpkg update
        ```

        Cette commande contacte le miroir choisi, télécharge les fichiers de métadonnées (CHECKSUMS.md5, MANIFEST.bz2, etc.) et vérifie les signatures GPG. Si elle se termine sans erreur, votre miroir est fonctionnel.

    3. **Installer les nouveaux paquets ajoutés à "Current" :**

        ```bash
        sudo slackpkg install-new
        ```

        Ceci installe les paquets qui ont été ajoutés à la distribution depuis votre dernière mise à jour.

    4. **Mettre à niveau les paquets installés vers leur dernière version :**

        ```bash
        sudo slackpkg upgrade-all
        ```

        Ceci met à jour tous les paquets installés sur votre système vers la version disponible sur le miroir.

    5. **Nettoyer les paquets obsolètes (optionnel mais recommandé) :**

        ```bash
        sudo slackpkg clean-system
        ```

        Ceci liste les paquets installés sur votre système qui ne font plus partie de la distribution "current" (ils ont été retirés ou remplacés). Vous pouvez choisir de les désinstaller pour garder un système propre.

* **LISEZ LE ChangeLog.txt !** C'est peut-être l'étape la plus importante pour un utilisateur de "current". **Avant** chaque `slackpkg upgrade-all`, consultez attentivement le fichier `/var/log/packages/ChangeLog.txt` (qui est mis à jour par `slackpkg update`). Il détaille tous les changements récents (paquets ajoutés, supprimés, mis à jour) et contient souvent des notes importantes de l'équipe Slackware sur d'éventuelles actions manuelles requises ou des changements majeurs. Ignorer le ChangeLog est le moyen le plus sûr de rencontrer des problèmes avec "current". Vous pouvez aussi le consulter en ligne sur le miroir que vous utilisez.

---

Ce guide fournit les étapes générales. Adaptez-les à votre matériel et à vos besoins spécifiques. Bonne installation !
