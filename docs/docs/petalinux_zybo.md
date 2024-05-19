# Linux sur Zybo Z7-20 avec PetaLinux

## Introduction

La Zybo Z7-20 est une carte de développement basée sur un FPGA Zynq-7000 de Xilinx, qui combine un processeur ARM Cortex-A9 bicœur et des ressources logiques programmables.

Ce guide a pour objectif de décrire l'installation et la configuration de Linux sur la Zybo Z7-20 en utilisant Vivado pour la génération du matériel et PetaLinux pour la création du système d'exploitation embarqué.


## Prérequis

- **Vivado** : Pour générer le fichier bitstream et le fichier XSA. [Télécharger Vivado](https://www.xilinx.com/support/download.html).
- **PetaLinux** : Pour générer le système de fichiers, le FSBL (First Stage Bootloader), U-Boot et le noyau Linux. [Guide d'installation PetaLinux](https://docs.amd.com/r/en-US/ug1144-petalinux-tools-reference-guide/Installation-Steps).

## Génération du Matériel avec Vivado

L'objectif est de générer un fichier XSA à partir du projet Vivado pour l'utiliser dans PetaLinux.

Nous utilisons le dépôt officiel de la Zybo Z7-20 disponible sur [GitHub](https://github.com/Digilent/Zybo-Z7) en suivant la documentation [Digilent FPGA Demo Git Repositories](https://digilent.com/reference/programmable-logic/documents/git?redirect=1). Le dépot n'est pas mis à jour pour la version 2023.2 de Vivado, mais Vivado est capable de mettre à jour les IPs automatiquement.


1. Cloner le dépôt :

    ```bash
    git clone --recursive https://github.com/Digilent/Zybo-Z7
    cd Zybo-Z7
    git submodule update --init --recursive
    git checkout remotes/origin/20/Petalinux/master
    git submodule update --init --recursive
    ```

2. Créer le projet Vivado en utilisant la commande suivante dans la console Tcl pour importer le projet :

    ```tcl
    set argv ""; source {local root repo}/hw/scripts/checkout.tcl
    ```

3. Générer le bitstream. Un bitstream est un fichier binaire contenant la configuration complète pour programmer un FPGA. Il décrit les connexions et les configurations des éléments logiques programmables en fonction de votre design matériel dans Vivado.

    !!! bug "Erreur"
        ```plaintext
        [IP_Flow 19-4965] IP ila_pixclk was packaged with board value 'digilentinc.com:zybo-z7-10:part0:1.0'. Current project's board value is 'digilentinc.com:zybo-z7-20:part0:1.1'. Please update the project settings to match the packaged IP.
        ```

        - **Solution** : Mettre à jour l'IP `ila_pixclk` pour correspondre à la valeur de la carte Zybo-Z7-20.

4. Mettre à jour l'IP **div2rgb** si nécessaire. (A VERIFIER ?)

5. Exporter le matériel en générant le fichier XSA (Hardware Specification Archive). Le fichier XSA contient toutes les informations nécessaires sur le design matériel, y compris les configurations des blocs IP, les contraintes de timing, les périphériques et les connexions internes. Il inclut également le bitstream généré.

Nous avons maintenant un fichier XSA que nous pouvons utiliser pour générer le système d'exploitation avec PetaLinux.

## Génération du Système d'Exploitation avec PetaLinux

### Création d'un Nouveau Projet

Générer un projet à partir du template Zynq :

```bash
petalinux-create --type project --template zynq --name petalinux_zybo
```

### Configuration du Projet

Configurer le projet avec le fichier XSA précédemment généré :

```bash
cd petalinux_zybo
petalinux-config --get-hw-description=<chemin_du_fichier_xsa>
```

### Paramètres à Configurer

1. **DTG Settings -> Kernel bootargs -> Add extra boot args** : Ajouter `rw` aux arguments de démarrage à cause d'une erreur dans les arguments générés par PetaLinux (par défaut `ro`).
2. **Image Packaging Configuration -> Root filesystem type** : Changer en `EXT4`.

### Compiler & Générer les Fichiers Nécessaires

1. Construire le projet :

    ```bash
    petalinux-build
    ```

    !!! bug "Erreur"
        ```plaintext
        [libtinfo.so.5: cannot open shared object file: No such file or directory]
        ```

        - **Solution** : Sur Ubuntu, installer la bibliothèque requise :

        ```bash
        sudo apt-get install libtinfo5
        ```
    
    !!! warning "Avertissement"
        ```plaintext
        The busybox:do_fetch sig is computed to be e69f899fec71b53e8a81fc4af8d446ffaed28c1f02b202fa8cb783dce7c0d2e4, but the sig is locked to 4dcaff8c51438430803be8fa00f31476d3041a4c2d22474848e638e2fe9ebaba in SIGGEN_LOCKEDSIGS_t-cortexa9t2hf-neon
        ```

        - **Note** : Cet avertissement dans la version 2023.2 (et non 2023.1) concernant `busybox:do_fetch sig` est normal et n'affecte pas le processus. [Support Xilinx](https://support.xilinx.com/s/article/000035704).

2. Générer l'image de démarrage :

    ```bash
    petalinux-package --boot --u-boot
    ```

3. Vous devriez avoir les fichiers suivants dans le répertoire `./images/linux` :

    ```plaintext
    BOOT.BIN
    boot.scr
    image.ub
    ```

### Root File System

Des distributions minimales Debian/Ubuntu sont disponibles sur [eewiki minfs](https://rcn-ee.com/rootfs/eewiki/minfs/). Téléchargez, par exemple, `debian-12.1-minimal-armhf-2023-08-22.tar.xz`.

Sans ces rootfs pré-construits, vous devriez préparer le vôtre en utilisant `debootstrap`.

### Préparation de la Carte SD

1. **Formater la Carte SD** avec les partitions nécessaires. ([Source](https://docs.amd.com/r/en-US/ug1144-petalinux-tools-reference-guide/Configuring-SD-Card-ext-File-System-Boot))

    - Aligner à 4MiB.
    - Première partition : "BOOT", Au moins 500 MB. Formater en FAT32. 4MiB libres avant la première partition.
    - Deuxième partition : "RootFS", Utiliser l'espace restant. Formater en EXT4.

2. **Copier les Images**. ([Source](https://docs.amd.com/r/en-US/ug1144-petalinux-tools-reference-guide/Booting-a-PetaLinux-Image-on-Hardware-with-SD-Card))

    - Copier `BOOT.BIN`, `boot.scr`, `image.ub` dans la partition boot.
    - Copier le `rootfs ` dans la partition rootfs puis décompresser.

### Démarrage de la Zybo Z7-20 (TODO)

1. Insérez la carte SD dans votre Zybo Z7-20 et configurez les jumpers pour démarrer depuis la carte SD.
2. Allumez la carte et surveillez le processus de démarrage via une connexion série (en utilisant minicom ou putty).

5. Lancer terminal série
Lancer un terminal série type gtkterm ou autre sur le bon port avec un baud rate de 115200, 8 bits et 1 bit de stop et aucun bit de parité
Pour la zybo, configurer la source de l'alim (USB ou WALL/secteur avec le JP6) et la source du boot (shunter avec le jumper les deux pins les plus à gauche de JP5). Des infos sont écrites sur le PCB dans tous les cas.
Mettre la carte SD dans la zybo.
Alimenter la Zybo de la manière souhaitée et brancher en usb au pc pour communiquer avec le terminal série.