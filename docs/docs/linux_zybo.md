# Installation de Linux sur la Zybo Z7-20 avec PetaLinux

## Introduction

La Zybo Z7-20 est une carte de développement basée sur un FPGA Zynq-7000 de Xilinx, qui combine un processeur ARM Cortex-A9 bicœur et des ressources logiques programmables.

Ce guide a pour objectif de décrire l'installation et la configuration de Linux sur la Zybo Z7-20 en utilisant Vivado pour la génération du matériel et PetaLinux pour la création du système d'exploitation embarqué.


## Prérequis

- **Vivado** : Pour générer le bitstream et le XSA. [Télécharger Vivado](https://www.xilinx.com/support/download.html).
- **PetaLinux** : Pour générer le système de fichiers, le FSBL (First Stage Bootloader), U-Boot et le noyau Linux. [Guide d'installation PetaLinux](https://docs.amd.com/r/en-US/ug1144-petalinux-tools-reference-guide/Installation-Steps).

## Génération du Matériel

L'objectif est de générer un fichier XSA à partir du projet Vivado pour l'utiliser dans PetaLinux.

!!! info "Qu'est-ce qu'un fichier XSA ?"
    Le fichier XSA contient toutes les informations nécessaires sur le design matériel, y compris les configurations des blocs IP, les contraintes de timing, les périphériques et les connexions internes. Il inclut également le bitstream généré.

Nous utilisons le dépôt officiel de la Zybo Z7-20 disponible sur [GitHub](https://github.com/Digilent/Zybo-Z7) en suivant le guide [Digilent FPGA Demo Git Repositories](https://digilent.com/reference/programmable-logic/documents/git?redirect=1). Ce dépôt contient une démonstration de la Zybo Z7-20 avec un design matériel complet et prêt à être utilisé.

!!! warning "Attention"
    Le dépot n'est pas mis à jour pour la version 2023.2 de Vivado, mais Vivado est capable de mettre à jour les IPs automatiquement.


1. Cloner le dépôt :

    ```bash
    git clone --recursive https://github.com/Digilent/Zybo-Z7
    cd Zybo-Z7
    git checkout remotes/origin/20/Petalinux/master
    git submodule update --init --recursive
    ```

2. Créer le projet Vivado en utilisant la commande suivante dans la console Tcl pour importer le projet :

    ```tcl
    set argv ""; source {local root repo}/hw/scripts/checkout.tcl
    ```
    Veiullez remplacer `{local root repo}` par le chemin absolu du dépôt cloné.

3. Générer le bitstream. Un bitstream est un fichier binaire contenant la configuration complète pour programmer un FPGA. Il décrit les connexions et les configurations des éléments logiques programmables en fonction de votre design matériel dans Vivado.

    !!! bug "Erreur"
        ```plaintext
        [IP_Flow 19-4965] IP ila_pixclk was packaged with board value 'digilentinc.com:zybo-z7-10:part0:1.0'. Current project's board value is 'digilentinc.com:zybo-z7-20:part0:1.1'. Please update the project settings to match the packaged IP.
        ```

        - **Solution Optionelle** : Mettre à jour l'IP `ila_pixclk` pour correspondre à la valeur de la carte Zybo-Z7-20.
        Cette erreur n'affecte pas la génération du bitstream ou le fonctionnement ultérieur.

5. Exporter le matériel en générant le fichier XSA (Hardware Specification Archive). 

Nous avons maintenant un fichier XSA que nous pouvons utiliser pour générer le système d'exploitation avec PetaLinux.

## Génération du Système d'Exploitation

### Petalinux

PetaLinux est un outil de développement d'OS embarqué qui simplifie le processus de création d'un système d'exploitation Linux embarqué pour les systèmes Zynq-7000, Zynq UltraScale+ MPSoC et MicroBlaze. L'objectif est d'utiliser le fichier XSA généré précédemment pour créer un système d'exploitation Linux embarqué pour la Zybo Z7-20.

1.  Créer un nouveau projet

    Générer un projet à partir du template Zynq :

    ```bash
    petalinux-create --type project --template zynq --name petalinux_zybo
    ```

1.  Configurer le projet

    Configurer le projet avec le fichier XSA précédemment généré :

    ```bash
    cd petalinux_zybo
    petalinux-config --get-hw-description=<chemin_du_fichier_xsa>
    ```

    Veillez à activer les options suivantes :

    - Image Packaging Configuration -> Root filesystem type : Changer en `EXT4`.
    - FPGA Manager -> FPGA Manager : Activer le FPGA Manager.
    
    !!! bug "Petalinux 2023.2"
        Une erreur est présente dans la version 2023.2 de PetaLinux au niveau des bootargs, `ro` est présent à la place de `rw`. 
        
        Une solution possible est de rajouter `rw` aux bootargs dans DTG Settings -> Kernel bootargs -> Add extra boot args.

    ??? tip "Définir l'adresse MAC dans Petalinux"
        Dans le menu de configuration Petalinux (`petalinux-config`), naviguez vers `Subsystem > Auto Hardware Settings > Ethernet Settings` pour définir l'adresse MAC de l'interface Ethernet. Vous avez deux principales options pour définir l'adresse MAC :

        1.  **Spécifier une adresse MAC statique :**
            - Entrez directement l'adresse MAC souhaitée dans le champ `Ethernet MAC address`.

        2.  **Activer la génération d'une adresse MAC aléatoire :**
            - Activez l'option `Randomise MAC address`. Cela vous permet de définir un modèle pour l'adresse MAC en utilisant le caractère `?` pour spécifier des valeurs aléatoires.
            - Exemple : `00:0a:35:00:??:??` – Ce modèle fixe les quatre premiers octets et rend les deux derniers octets aléatoires.

        **Remarque :** Si vous avez configuré une adresse MAC dans U-Boot, l'adresse MAC de l'environnement U-Boot aura la priorité sur la configuration Petalinux.

        **Remarque :** Lors de l'utilisation de l'option `Randomise MAC address`, l'interface se verra attribuer une adresse MAC lors du premier démarrage et conservera cette adresse pour les démarrages suivants.

        Pour plus d'informations détaillées, consultez le [Guide de Référence des Outils Petalinux de AMD (UG1144)](https://docs.amd.com/r/en-US/ug1144-petalinux-tools-reference-guide/Building-Device-Tree-Overlays-for-PS).


2.  Configurer le noyau

    ```bash
    petalinux-config -c kernel
    ```

    Veillez à activer les options suivantes du noyau :

    - File systems -> Ext4 POSIX Access Control Lists : Activer.
            
    - File systems -> Kernel automounter support (supports v3, v4 and v5) : Activer.

    - Executable file formats -> Kernel support for MISC binaries : Activer.

3. Compiler & Générer les Fichiers Nécessaires

    1. compiler le projet :

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

Des distributions minimales Debian/Ubuntu sont disponibles sur [eewiki minfs](https://rcn-ee.com/rootfs/eewiki/minfs/). 
Téléchargez, par exemple, `debian-12.1-minimal-armhf-2023-08-22.tar.xz`. Veillez utiliser la version `armhf`.
!!! note "Note"
    Il est possible de générer un RFS personnalisé nottament avec `debootstrap` ou `buildroot`.


### Préparation de la Carte SD

1. **Formater la Carte SD** avec les partitions nécessaires. ([Source](https://docs.amd.com/r/en-US/ug1144-petalinux-tools-reference-guide/Configuring-SD-Card-ext-File-System-Boot))

    - Aligner à 4MiB.
    - Première partition : "BOOT", Au moins 500 MB. Formater en FAT32. 4MiB libres avant la première partition.
    - Deuxième partition : "RootFS", Utiliser l'espace restant. Formater en EXT4.

    ??? tip "Procédure pour formater une carte SD avec `fdisk`"
        **Étape 1**: Lister les devices disponibles

        D'abord, lister tous les devices de stockage disponibles pour identifier le device correspondant à la carte SD.

        ```bash
        sudo fdisk -l
        ```

        Cherchez votre carte SD dans la liste des devices. Supposons qu'il s'agit de `/dev/mmcblk0` dans cet exemple.

        **Étape 2**: Démarrer `fdisk` pour partitionner la carte SD

        Démarrez `fdisk` sur le device de la carte SD.

        ```bash
        sudo fdisk /dev/mmcblk0
        ```

        **Étape 3**: Ajouter une nouvelle partition

        1. Ajouter une nouvelle partition:

               ```
               Command (m for help): n
               ```

               - Choisir le type de partition : primaire (`p`).
               - Numéro de partition : 1.
               - Premier secteur : 8192.
               - Dernier secteur : +500M (pour créer une partition de 500 MiB).

               ```
               Partition type
                  p   primary (0 primary, 0 extended, 4 free)
               Select (default p): p
               Partition number (1-4, default 1): 1
               First sector (2048-15523839, default 2048): 8192
               Last sector, +/-sectors or +/-size{K,M,G,T,P} (8192-15523839, default 15523839): +500M
               ```

        2. Ajouter une seconde partition:

               ```
               Command (m for help): n
               ```

               - Choisir le type de partition : primaire (`p`).
               - Numéro de partition : 2.
               - Premier secteur : utiliser le meme secteur que la fin de la première partition afin de laisser fdisck calculer l'espace restant.
               - Dernier secteur : laisser par défaut pour utiliser l'espace restant.

               ```
               Command (m for help): n
                Partition type
                   p   primary (1 primary, 0 extended, 3 free)
                   e   extended (container for logical partitions)
                Select (default p): p
                Partition number (2-4, default 2): 2
                First sector (2048-15523839, default 2048): 8192

                Sector 8192 is already allocated.
                First sector (1032192-15523839, default 1032192): [Appuyer sur Entrée pour prendre par défaut]
                Last sector, +/-sectors or +/-size{K,M,G,T,P} (1032192-15523839, default 15523839): : [Appuyer sur Entrée pour prendre par défaut]
               ```

        **Étape 4**: Écrire les changements

        Écrire les modifications sur le disque et quitter `fdisk`:

        ```
        Command (m for help): w
        ```

        **Étape 5**: Formater les partitions

        1.  Lister les partitions pour confirmer les nouvelles partitions:

            ```bash
            sudo fdisk -l
            ```

        2.  Formater la première partition en FAT32:

            ```bash
            sudo mkfs.fat -F 32 /dev/mmcblk0p1
            ```

        3.  Formater la seconde partition en ext4:

            ```bash
            sudo mkfs.ext4 /dev/mmcblk0p2
            ```

    ??? info "Exemple carte SD préparée"
        ```plaintext
        Numéro de partition (1,2, 2 par défaut) : 1

             Device: /dev/mmcblk0p1
              Start: 8192
                End: 1032191
            Sectors: 1024000
          Cylinders: 16001
               Size: 500M
                 Id: 83
               Type: Linux
        Start-C/H/S: 128/0/1
          End-C/H/S: 1023/3/16

        Commande (m pour l'aide) : i
        Numéro de partition (1,2, 2 par défaut) : 2

             Device: /dev/mmcblk0p2
              Start: 1032192
                End: 60751871
            Sectors: 59719680
          Cylinders: 933121
               Size: 28,5G
                 Id: 83
               Type: Linux
        Start-C/H/S: 1023/3/16
          End-C/H/S: 1023/3/16
        ```

        ```plaintext
        lsblk -fs
        NAME      FSTYPE FSVER LABEL                 UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
        sda1      vfat   FAT32                       84C0-4ABA                             579,8M     3% /boot/efi
        └─sda
        sda2      ext4   1.0                         4187e4a5-99f2-481b-b7fd-a80cf2a1d054    601M    31% /boot
        └─sda
        sda3      btrfs        fedora_localhost-live b630913f-76fe-45d9-bb4f-62c1e6005aeb  172,4G    22% /home
        │                                                                                                /
        └─sda
        sr0
        mmcblk0p1 vfat   FAT32                       EC04-463A
        └─mmcblk0
        mmcblk0p2 ext4   1.0                         29000ca8-9682-4bc6-ab5a-9963948fa3a5
        └─mmcblk0
        ```

        ```plaintext
        lsblk -a
        NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
        sda           8:0    0 223,6G  0 disk 
        ├─sda1        8:1    0   600M  0 part /boot/efi
        ├─sda2        8:2    0     1G  0 part /boot
        └─sda3        8:3    0   222G  0 part /home
                                              /
        sr0          11:0    1  1024M  0 rom
        mmcblk0     179:0    0    29G  0 disk 
        ├─mmcblk0p1 179:1    0   500M  0 part 
        └─mmcblk0p2 179:2    0  28,5G  0 part 
        zram0       252:0    0   7,6G  0 disk [SWAP]
        ```

2. **Copier les Images**. ([Source](https://docs.amd.com/r/en-US/ug1144-petalinux-tools-reference-guide/Booting-a-PetaLinux-Image-on-Hardware-with-SD-Card))

    - Copier `BOOT.BIN`, `boot.scr`, `image.ub` dans la partition boot.
    - Copier le `rootfs ` dans la partition rootfs puis décompresser.

## Démarrage de la Zybo Z7-20 

1.  Insérez la carte SD dans votre Zybo Z7-20 et configurez la carte pour démarrer depuis la carte SD. ([Digilent Zybo Z7-20 Reference Manual](https://digilent.com/reference/programmable-logic/zybo-z7/reference-manual) section 2.1)

    - Insérez la carte microSD dans le connecteur J4.
    - Attachez une source d'alimentation à la Zybo Z7 et sélectionnez-la en utilisant JP6.
    - Placez un seul cavalier sur JP5, reliant les deux broches les plus à gauche (étiquetées "SD").
    - Allumez la carte. La carte démarrera maintenant l'image sur la carte microSD.

2.  Allumez la carte et surveillez le processus de démarrage via une connexion série.

    Lancer un terminal série type gtkterm ou autre sur le bon port avec un baud rate de 115200, 8 bits et 1 bit de stop et aucun bit de parité

3.  vous devriez voir le démarrage de Linux sur votre Zybo Z7-20. Have fun !

!!! failure "Problèmes Connus lors du Démarrage"
    ```plaintext
    Starting init: /bin/sh exists but couldn't execute it (error -5)
    Kernel panic - not syncing: No working init found.  Try passing init= option to kernel. See Linux Documentation/admin-guide/init.rst for guidance.
    ```
    - **Solution** : Le RootFS est corrompu. Refaites la copie du RootFS sur la carte SD en vérifiant le sha256sum.