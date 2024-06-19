# Reprogrammation FPGA et application UIO pour PmodENC sous Linux

## Introduction

Ce projet a pour but de récupérer les données du compteur de notre [bloc IP personnalisé](./pmodenc_ip.md) en passant par le driver UIO de Linux sur la carte Zybo Z7-20, et ce en reprogrammant le FPGA. Le Zynq n'a donc pas l'encodeur et son bloc IP ni dans le FPGA ni dans le [Device Tree](./device_tree.md), un [Device Tree Overlay](./device_tree.md#device-tree-overlay) est donc utilisé en plus d'un nouveau bitstream à implémenter avec le [FPGA Manger](./fpga_manager.md). Avoir Linux sur la carte est un prérequis, vous pouvez suivre notre [guide](./linux_zybo.md) pour déployer Debian dessus ou utiliser un autre RFS.

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

2.  Accéder la configuration du kernel avec la commmande `petalinux-config -c kernel` dans le dossier du projet.

3.  Activer les drivers UIO.
   
    Les options à activer sont :

    - Device Drivers -> Userspace I/O drivers -> Userspace I/O platform driver with generic IRQ handling
    - Device Drivers -> Userspace I/O drivers -> Userspace platform driver with generic irq and dynamic memory 

4.  Sauvegarder la configuration et quitter `petalinux-config`.

5.  Build le kernel avec `petalinux-build -c kernel`.

6.  Transférer le kernel réalisé vers la carte.

7.  Ouvrir le dossier contenant les artéfacts de build PetaLinux qui se trouve généralement dans `/tmp/`.
   
8.  Se diriger dans le répertoire contenant les modules kernel et les drivers : `work/zynq_generic_7z020-xilinx-linux-gnueabi/linux-xlnx/6.1.30-xilinx-v2023.2+gitAUTOINC+a19da02cf5-r0/image/lib/modules/`. Le nom des dossiers peut varier mais suit globalement toujours la même logique.

9.  Déplacer le dossier `6.1.30-xilinx-v2023.2`, dont le nom peut varier selon la version du kernel et de PetaLinux, dans le dossier `lib/modules` de votre RFS.
    
10.  La carte est désormais prête au niveau des drivers.

## Bitstream 

### Introduction


