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

* **Mises à jour :** "Current" change souvent. Mettez à jour régulièrement.
    1. Configurez `slackpkg` : éditez `/etc/slackpkg/mirrors` et décommentez **un seul** miroir "current" proche de vous (ex: Montpellier -> un miroir français).
    2. Exécutez (en tant que `root` ou via `sudo`) :

        ```bash
        slackpkg update
        slackpkg install-new
        slackpkg upgrade-all
        slackpkg clean-system
        ```

* **LISEZ LE ChangeLog.txt !** Avant chaque `slackpkg upgrade-all`, lisez impérativement le fichier `/var/log/packages/ChangeLog.txt` (ou celui sur le miroir). Il détaille les changements récents et indique s'il y a des étapes manuelles ou des précautions particulières à prendre. C'est **vital** pour la maintenance de "current".

---

Ce guide fournit les étapes générales. Adaptez-les à votre matériel et à vos besoins spécifiques. Bonne installation !
