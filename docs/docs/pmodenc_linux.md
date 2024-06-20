# Reprogrammation FPGA et application UIO pour PmodENC sous Linux

## Introduction

Ce projet vise à récupérer les données du compteur de notre [bloc IP personnalisé](./pmodenc_ip.md) via le driver UIO de Linux sur la carte Zybo Z7-20, en reprogrammant le FPGA. Le Zynq n'inclut pas initialement l'encodeur et son bloc IP dans le FPGA ni dans le [Device Tree](./device_tree.md). Pour pallier cela, un [Device Tree Overlay](./device_tree.md#device-tree-overlay) est utilisé, accompagné d'un nouveau bitstream à implémenter avec le [FPGA Manager](./fpga_manager.md). **Avoir Linux installé sur la carte est un prérequis**. Vous pouvez suivre notre [guide](./linux_zybo.md) pour déployer Debian ou utiliser un autre RFS.

Ce projet est disponible sur [GitHub](https://github.com/TER-Zybo/PmodENC_Linux).

---

## Drivers

### Introduction

Pour faire fonctionner l'encodeur via UIO, il est nécessaire de disposer du driver approprié. Cela implique d'apporter quelques modifications initiales dans PetaLinux, puis de transférer les modules du kernel vers le RFS.

!!! info "Custom RFS"
    Le RFS fourni par PetaLinux lors de la génération du projet contient déjà les drivers UIO. Cette partie ne s'applique que si vous utilisez un RFS autre que celui fourni par PetaLinux, comme vu dans notre guide. Cependant assurez-vous d'avoir les options de l'étape 3 activées.

### Prérequis

- Projet PetaLinux avec son XSA

> Pour plus d'informations sur la génération du projet PetaLinux, veuillez consulter notre [guide](./linux_zybo.md).

### Étapes

1.  Modifier la configuration PetaLinux pour conserver le dossier `linux-xlnx` contenant les différents artéfacts du kernel.
    
    - Ajouter `RM_WORK_EXCLUDE += "linux-xlnx"` dans le fichier `petalinuxbsp.conf` disponible dans le dossier `project-spec/meta-user/conf/` du projet PetaLinux.

    ??? note "Les artéfacts du kernel"
        Les artefacts du kernel sont les fichiers générés lors de la compilation du noyau Linux, tels que les modules du kernel, les fichiers de configuration, et d'autres fichiers nécessaires au bon fonctionnement du système d'exploitation.

2.  Accéder à la configuration du kernel et activer les drivers UIO.

    ```
    petalinux-config -c kernel
    ```

    Veillez à activer les options suivantes :

    - Device Drivers -> Userspace I/O drivers -> Userspace I/O platform driver with generic IRQ handling
    - Device Drivers -> Userspace I/O drivers -> Userspace platform driver with generic irq and dynamic memory 

3.  Modifier les bootargs du kernel.

    ```
    petalinux-config
    ```

    - DTG Settings -> Kernel Bootargs -> Add extra boot args : Ajouter `uio_pdrv_genirq.of_id=generic-uio`.

    !!! tip "TMPDIR emplacement"
        Nous vous recommandons de noter le chemin du dossier temporaire créé par PetaLinux, car il sera nécessaire pour récupérer les modules du kernel.
        Pour connaître le chemin du dossier temporaire créé par PetaLinux, allez dans :
        
        - Yocto Settings -> TMPDIR Location -> TMPDIR Location

4.  Build le kernel. 
    
    ```
    petalinux-build -c kernel
    ```

5.  Transférer le kernel vers la partition de BOOT de la carte SD.

    > plus d'informations sur le transfert du kernel [ici](./linux_zybo.md#preparation-de-la-carte-sd).


10. Copier le dossier des modules du kernel dans le RFS.

    Accéder au TMPDIR généré par petalinux et se rendre dans le répertoire contenant les artefacts du kernel.

    ```
    cd [TMPDIR]/work/zynq_generic_7z020-xilinx-linux-gnueabi/linux-xlnx/6.1.30-xilinx-v2023.2+gitAUTOINC+a19da02cf5-r0/image/lib/modules/
    ```

    Copier le dossier `6.1.30-xilinx-v2023.2` dans le dossier `lib/modules` de votre RFS. Le nom du dossier peut varier en fonction de la version du kernel et de PetaLinux.

    !!! warning "Chemin des modules et nom du dossier"
        Le chemin des modules et le nom du dossier peuvent varier en fonction de la version du kernel et de PetaLinux.

13.  Félicitations, les drivers UIO sont désormais disponibles dans votre RFS.

---

## Bitstream 

### Introduction

Pour reprogrammer le FPGA, vous devez disposer d'un bitstream comprenant le block design utilisé précédemment pour générer le projet Petalinux, ainsi que les blocs IP et modules RTL que vous souhaitez ajouter. 

Dans le cadre de notre projet, le seul bloc IP ajouté est [PetaENC](./pmodenc_ip.md).

### Prérequis

La procédure décrite ci-dessous suppose que :

1. Le block design de base est celui récupéré lors de l'étape [Génération du Matériel](./linux_zybo.md) de notre guide Linux.
2. Le bloc IP ajouté est [PetaENC](./pmodenc_ip.md) disponible sur [github](https://github.com/TER-Zybo/PetaENC_IP).

!!! tip "Pret à l'emploi"
    Si vous souhaitez utiliser le projet avec le block design final, vous pouvez le récupérer sur [github](https://github.com/TER-Zybo/PmodENC_Linux).


### Étapes

1.  Ouvrir le projet `hw_with_enc` dans Vivado, disponible en clonant le [repository git](https://github.com/TER-Zybo/PmodENC_Linux).

2.  S'assurer que le bloc IP personnalisé est bien dans le catalogue, voir [Ajout du Bloc IP à Vivado](./pmodenc_ip.md#ajout-du-bloc-ip-a-vivado) pour plus d'informations.

3.  Réaliser la synthèse, l'implémentation et la génération du bitstream.

4.  Exporter le bitstream en faisant clic-droit sur l'implémentation dans la section `Project Manager > Design Runs` puis `Export Bitstream File...`.

5.  Suivre les indications de la section [Génération d'un Fichier BIN à partir d'un Fichier BIT](./fpga_manager.md#generation-dun-fichier-bin-a-partir-dun-fichier-bit) de la documention sur le [FPGA Manager](./fpga_manager.md) afin de transformer le BIT en BIN.

6.  Transférer le BIN sur l'envirronement linux de la Zybo.

7.  Félicitations, le bitstream est prêt à être utilisé pour reprogrammer le FPGA.

---

## Overlay

### Introduction

Tout comme le bitstream est nécessaire pour reprogrammer le FPGA, le [Device Tree Overlay](./device_tree.md#device-tree-overlay) est nécessaire afin que le kernel Linux puisse interagir avec notre [bloc IP personnalisé](./pmodenc_ip.md).

Le DTO utilisé ici est spécifique à notre bloc IP et est expliqué plus en détail dans la section [Exemple DTO pour un encodeur](./device_tree.md#exemple-dto-pour-un-encodeur) dans la documentation sur les [DT](./device_tree.md).

### Prérequis

- Fichier source `.dtso` du DTO, disponible sur [github](https://github.com/TER-Zybo/PmodENC_Linux) du projet.
- Device Tree Compiler (DTC) pour compiler le DTO. Disponible dans vitis (`Vitis/2023.1/bin/` pour la versin 2023.1) ou depuis le [repository officiel](https://git.kernel.org/pub/scm/utils/dtc/dtc.git)
  
### Étapes

!!! note "Correspondance des noms"
    Assurez-vous que le nom du fichier `.bit.bin` généré dans la section précédente correspond à celui utilisé dans le DTO. Veillez modifier le DTO si nécessaire.

1.  Suivre la procédure de [compilation d'un DTO](./device_tree.md#procedure_1).

2.  Transférer le `.dtbo` résultant sur l'environnement linux de la Zybo.

3. Félicitations, le DTO est prêt à être utilisé pour reprogrammer le FPGA.


---

## Application

### Introduction

Une fois que les drivers, le bitstream au format BIN, et le DTO compilé sont disponibles sur la Zybo, la reprogrammation du FPGA peut être effectuée. Cependant, il est également nécessaire de disposer d'une application pour tester le bon fonctionnement de notre encodeur. Une application simple utilisant le driver UIO est disponible dans le dossier `app` du [repository git](https://github.com/TER-Zybo/PmodENC_Linux) de notre projet.

Cette application doit être compilée pour l'architecture cible, en l'occurrence **armhf**. Il est donc nécessaire d'utiliser un cross-compiler pour effectuer cette compilation.

### Prérequis

- Fichier `.c` de l'application `check_uio_value`, disponible dans `app` dans notre [repository git](https://github.com/TER-Zybo/PmodENC_Linux).
- Cross-compiler GCC armhf, tel que celui disponible sur le [site officiel de développement ARM](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads), assurez-vous de télécharger une version ciblant Arch32 GNU/Linux avec hard float (arm-none-linux-gnueabihf).

### Étapes

1.  Compiler l'application `check_uio_value` avec cross-compiler GCC

    ```
    arm-none-linux-gnueabihf-gcc -o check_uio_value check_uio_value.c
    ``` 

2.  Transférer l'application résultante sur l'environnement linux de la Zybo.

3.  La rendre éxecutable avec `chmod +x check_uio_value`.

4.  Félicitations, vous avez désormais tout les éléments nécessaires pour reprogrammer le FPGA et tester l'encodeur.

---

## Reprogrammation et test de l'encodeur

### Introduction

Tous les élements nécessaires sont désormais réunis, il ne manque plus qu'à réaliser la reprogrammation et tester l'encodeur avec l'application précédemment compilée.

### Prérequis

- [Drivers UIO](#drivers)
- [Bitstream BIN](#bitstream)
- [DTO compilé](#overlay)
- [Application compilé](#application)

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



