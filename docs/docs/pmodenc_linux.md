# Reprogrammation FPGA et application UIO pour PmodENC sous Linux

## Introduction

Ce projet a pour but de récupérer les données du compteur de notre [bloc IP personnalisé](./pmodenc_ip.md) en passant par le driver UIO de Linux sur la carte Zybo Z7-20, et ce en reprogrammant le FPGA. Le Zynq n'a donc pas l'encodeur et son bloc IP ni dans le FPGA ni dans le [Device Tree](./device_tree.md), un [Device Tree Overlay](./device_tree.md#device-tree-overlay) est donc utilisé en plus d'un nouveau bitstream à implémenter avec le [FPGA Manager](./fpga_manager.md). Avoir Linux sur la carte est un prérequis, vous pouvez suivre notre [guide](./linux_zybo.md) pour déployer Debian dessus ou utiliser un autre RFS.

Ce projet est disponible sur [GitHub](https://github.com/TER-Zybo/PmodENC_Linux).

## Drivers

Pour faire fonctionner l'encodeur par UIO, il est nécessaire d'avoir le driver correspondant. Quelques modifications sont à faire dans PetaLinux initialement puis un transfert des modules kernel vers le RFS est à réaliser.

!!! info "Custom RFS"
    Cette partie ne s'applique que si vous utilisez PetaLinux avec un RFS autre que celui fourni, comme vu dans notre guide, et/ou si le RFS utilisé ne dispose pas des drivers UIO.

    Si vous utilisez l'image proposée par PetaLinux, assurez-vous d'avoir les options de l'étape 3 activées, vous pouvez ignorer le transfert dans le RFS et transférer votre image normalement.

### Prérequis

- Projet PetaLinux avec son XSA 

### Étapes

1.  Changer la configuration PetaLinux afin de garder le dossier `linux-xlnx` contenant les différents artéfacts du kernel.
   
    Il suffit pour cela de rajouter `RM_WORK_EXCLUDE += "linux-xlnx"` dans le fichier `[PETALINUX_PROJECT]/project-spec/meta-user/conf/petalinuxbsp.conf`.

2.  Accéder à la configuration du kernel avec la commmande `petalinux-config -c kernel` dans le dossier du projet.

3.  Activer les drivers UIO.
   
    Les options à activer sont :

    - Device Drivers -> Userspace I/O drivers -> Userspace I/O platform driver with generic IRQ handling
    - Device Drivers -> Userspace I/O drivers -> Userspace platform driver with generic irq and dynamic memory 

4.  Quitter `petalinux-config -c kernel` et accéder à la configuration PetaLinux avec `petalinux-config` dans le dossier du projet.

5.  Noter le dossier temporaire créé par PetaLinux, le chemin de ce répertoire se trouve en allant dans :
    
    - Yocto Settings -> TMPDIR Location -> TMPDIR Location

6.  Toujours dans `petalinux-config`, modifier les bootargs du kernel en rajoutant `uio_pdrv_genirq.of_id=generic-uio` dans :
    
    - DTG Settings -> Kernel Bootargs -> Add extra boot args

7.  Sauvegarder la configuration et quitter `petalinux-config`.

8.  Build le kernel avec `petalinux-build -c kernel`.

9.  Transférer le kernel `image.ub` situé dans `[PETALINUX_PROJECT]/images/linux` vers la partition de boot de votre Zybo.

10. Ouvrir le dossier temporaire noté précédemment.
   
11. Se diriger dans le répertoire contenant les modules kernel et les drivers : `work/zynq_generic_7z020-xilinx-linux-gnueabi/linux-xlnx/6.1.30-xilinx-v2023.2+gitAUTOINC+a19da02cf5-r0/image/lib/modules/`. Le nom des dossiers peut varier mais suit globalement toujours la même logique.

12. Déplacer le dossier `6.1.30-xilinx-v2023.2`, dont le nom peut varier selon la version du kernel et de PetaLinux, dans le dossier `lib/modules` de votre RFS.
    
13.  La carte est désormais prête au niveau des drivers.

## Bitstream 

### Introduction

Pour reprogrammer le FPGA, il est nécessaire d'avoir un bitstream contenant à la fois le BD de base, celui contenant le PS, de votre Zybo en plus des blocs IP et modules RTL que vous désirez rajouter. Ici, le seul bloc IP rajouté à l'occasion de notre projet est le [nôtre](./pmodenc_ip.md), contenant un compteur 4 bits fonctionnant grâce à l'encodeur. La suite de cette partie part donc du principe que seul ce bloc IP est rajouté et que le BD de base est celui du projet récupéré durant l'étape 2 de notre guide [Linux](./linux_zybo.md#generation-du-materiel). 

Il est par ailleurs considéré que le projet avec le BD final est déjà réalisé, vous pouvez récupérer celui que nous utilisons sur le [repository git](https://github.com/TER-Zybo/PmodENC_Linux) de ce projet.

### Prérequis

- Projet avec le BD final, disponible sur le [repository git](https://github.com/TER-Zybo/PmodENC_Linux)
- Bloc IP personnalisé
- Vivado 2023.1
- Vitis 2023.1

### Étapes

1.  Ouvrir le projet `hw_with_enc` dans Vivado, disponible en clonant le [repository git](https://github.com/TER-Zybo/PmodENC_Linux).

2.  S'assurer que le bloc IP personnalisé est bien dans le catalogue, voir [Ajout du Bloc IP à Vivado](./pmodenc_ip.md#ajout-du-bloc-ip-a-vivado) dans le guide sur le [bloc IP](./pmodenc_ip.md).

3.  Réaliser la synthèse, l'implémentation et la génération du bitstream.

4.  Exporter le bitstream dans un endroit au choix en faisant clic-droit sur l'implémentation dans `Design Runs` dans la section `Project Manager` puis `Export Bitstream File...`.

5.  Suivre les indications de la section [Génération d'un Fichier BIN à partir d'un Fichier BIT](./fpga_manager.md#generation-dun-fichier-bin-a-partir-dun-fichier-bit) de la documention sur le [FPGA Manager](./fpga_manager.md) afin de transformer le BIT en BIN.

6.  Transférer le BIN sur la Zybo, dans le dossier personnel `/home/[NOM D'UTILISATEUR]` par exemple.

7.  Renommez le BIN généré dans la section [Bitstream](#bitstream) en `enc_wrapper.bit.bin` étant donné que le DTO pointe vers ce nom de fichier.

## Overlay

### Introduction

Tout comme le bitstream est nécessaire pour reprogrammer le FPGA, le [Device Tree Overlay](./device_tree.md#device-tree-overlay) est nécessaire afin que le kernel Linux puisse interagir avec notre [bloc IP personnalisé](./pmodenc_ip.md).

Le DTO utilisé ici est spécifique à notre bloc IP et est expliqué plus en détail dans la section [Exemple DTO pour un encodeur](./device_tree.md#exemple-dto-pour-un-encodeur) dans la documentation sur les [DT](./device_tree.md).

### Prérequis

- Fichier source du DTO en .dtso, disponible sur le [repository git](https://github.com/TER-Zybo/PmodENC_Linux) du projet ou dans la section [Exemple DTO pour un encodeur](./device_tree.md#exemple-dto-pour-un-encodeur) dans la documentation sur les [DT](./device_tree.md).
- DTC, fourni avec Vitis, dans `Vitis/2023.1/bin/` pour la version 2023.1 par exemple, ou peut être téléchargé depuis le repository officiel [ici](https://git.kernel.org/pub/scm/utils/dtc/dtc.git).
  
### Étapes

1.  Suivre la procédure de [compilation d'un DTO](./device_tree.md#procedure-1).

2.  Transférer le .dtbo résultant sur la Zybo, dans le dossier personnel `/home/[NOM D'UTILISATEUR]` par exemple.

## Application

### Introduction

Une fois que les drivers, le bitstream en format BIN et le DTO compilé sont disponibles sur la Zybo, la reprogrammation est possible. Toutefois, une application pour tester le bon fonctionnement de notre encodeur est toujours nécessaire. Une simple application utilisant le driver UIO est donc disponible dans le [repository git](https://github.com/TER-Zybo/PmodENC_Linux) de notre projet dans le dossier `app`.

Toutefois, l'application doit être compilé pour l'architecture visée, en l'occurence `armhf`. L'utilisation d'un cross-compiler est donc nécessaire.

### Prérequis

- Fichier .c de l'application `check_uio_value`, disponible dans `app` dans notre [repository git](https://github.com/TER-Zybo/PmodENC_Linux).
- Cross-compiler GCC `armhf`, tel que celui disponible sur le [site officiel de développement ARM](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads), assurez-vous de télécharger une version ciblant AArch32 GNU/Linux avec hard float (arm-none-linux-gnueabihf).

### Étapes

1.  Compiler l'application `check_uio_value` avec votre cross-compiler GCC, par exemple `arm-none-linux-gnueabihf-gcc -o check_uio_value check_uio_value.c` dans le cas où l'on utilise celui proposé précédemment.

2.  Transférer l'application résultante sur la Zybo, dans le dossier personnel `/home/[NOM D'UTILISATEUR]` par exemple.

3.  La rendre éxecutable en faisant `chmod +x check_uio_value`.

## Reprogrammation et test de l'encodeur

Tous les élements nécessaires sont désormais réunis, il ne manque plus qu'à réaliser la reprogrammation et tester l'encodeur avec l'application précédemment compilée.

### Prérequis

- [Drivers](#drivers)
- [Bitstream](#drivers)
- [Overlay](#overlay)
- [Application](#application)

### Étapes

1.  Suivre les étapes de reprogrammation `Avec DTO` de la section [Mise à Jour du FPGA depuis Linux](./fpga_manager.md#mise-a-jour-du-fpga-depuis-linux) de notre guide [FPGA Manager](./fpga_manager.md).

2.  Une fois la reprogrammation réalisée, brancher l'encodeur dans le connecteur JE de la carte Zybo Z7-20.

3.  Lancer l'application `check_uio_value` avec la commande `check_uio_value 0 0x0 0x1000 polling`.

    - La première valeur `0` représente le numéro de device UIO visible dans `/dev/`, donc ici `/dev/uio0`.
    - La deuxième valeur représente l'offset de l'adresse à check, ici `0x0` étant donné que c'est le registre 0 de notre bloc IP à vérifier.
    - La troisième valeur représente la taille de la mémoire allouée au device, donc ici `0x1000`.
    - L'option `polling` permet d'afficher à l'écran la valeur du registre toutes les secondes.

4.  Tourner l'encodeur et voir la valeur du compteur augmenter. Vous pouvez appuyer sur l'encodeur pour remettre le codeur à 0.

5.  Faire ++ctrl+c++ pour quitter l'application.



