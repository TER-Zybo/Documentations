# Programmation FPGA depuis Linux avec FPGA Manager

## Introduction

Les FPGA (Field-Programmable Gate Arrays) sont des circuits intégrés reprogrammables offrant une grande flexibilité pour diverses applications. Pour programmer ces dispositifs, des fichiers spécifiques sont utilisés : les fichiers BIT et BIN. Ce guide vous explique comment générer un fichier BIN à partir d'un fichier BIT et l'utiliser pour programmer votre FPGA en utilisant l'outil FPGA Manager sous Linux.

## Les fichiers nécéssaires pour la programmation FPGA

### Qu'est-ce qu'un Bitstream ?

Un bitstream est un fichier qui contient les informations nécessaires pour configurer un FPGA. Il détermine comment les ressources logiques et les interconnexions du FPGA doivent être programmées pour effectuer des tâches spécifiques. Les bitstreams peuvent être générés à l'aide d'outils de conception FPGA comme Vivado.

### Différences entre Fichiers BIT et BIN

| Caractéristique      | Fichiers BIT (Bitstream)                                              | Fichiers BIN                                                |
|----------------------|-----------------------------------------------------------|-------------------------------------------------------------|
| **Contenu**          | Inclut les informations nécessaires pour programmer les ressources logiques et les interconnexions du FPGA. | Versions binaires du bitstream de configuration. Compatibles avec les outils de gestion FPGA sous Linux. |
| **Génération**       | Générés par des outils de synthèse et de placement/routage comme `Vivado`. | Générés à partir des fichiers BIT en utilisant l'outil `bootgen`.

!!! note "Pourquoi utiliser un fichier BIN ?"
    Les fichiers BIN sont utilisés pour charger les bitstreams dans les FPGA depuis un environnement Linux. Ils sont compatibles avec les outils de gestion FPGA comme FPGA Manager. Il n'est pas possible de charger directement un fichier BIT dans un FPGA depuis Linux.

### Génération d'un Fichier BIN à partir d'un Fichier BIT

bootgen est un outil fourni par Xilinx qui permet de générer des fichiers BIN à partir de fichiers BIF (Boot Image Format). L'outil est fourni avec Vitis de base et n'est pas disponible directement en open-source. Il se situe généralement dans le dossier d'installation de Vitis, par exemple avec la version 2023.1 `Vitis/2023.1/bin/`.

Pour générer un fichier BIN à partir d'un fichier BIT, nommé ici `project.bit`, suivez ces étapes :

1. Créez un fichier de configuration BIF nommé `project.bif` dans le même répertoire que votre fichier `project.bit` avec le contenu suivant :

    ```plaintext title="project.bif"
    all:
    {
        project.bit /* Nom du fichier Bitstream */
    }
    ```

    > Plus d'informations sur le format BIF sont disponibles dans la [documentation Xilinx](https://docs.amd.com/r/en-US/ug1400-vitis-embedded/Boot-Image-Format-BIF).

2. Utilisez la commande `bootgen`, situé par exemple avec la version 2023.1 dans `Vitis/2023.1/bin/`, pour générer le fichier BIN dans le même répertoire que le BIF :

    ```bash
    bootgen -image project.bif -arch zynq -process_bitstream bin
    ```

!!! warning "NE PAS UTILISER VIVADO !"
    Vivado génère des fichiers BIN qui ne sont pas compatibles avec le chargement du FPGA depuis Linux. Pour générer un fichier BIN compatible, utilisez `bootgen`.

    ```plaintext	
    Paramètres -> Bitstream -> -bin_file : activer
    ```

    Le fichier sera généré dans `proj_name/proj_name.runs/impl_1` ne sera pas compatible avec le chargement du FPGA depuis Linux.

    l'utilisation de `bootgen` est nécessaire pour générer un fichier BIN compatible.

## FPGA Manager

### Introduction à FPGA Manager

FPGA Manager est une interface du noyau Linux qui permet de charger et de gérer les bitstreams des FPGA. Il simplifie l'interaction avec le FPGA en exposant des fichiers virtuels à travers le système de fichiers Linux, ce qui facilite la reconfiguration et la mise à jour des FPGA.

Composants Principaux du FPGA Manager

- **flags** : Ce fichier permet de définir des options pour le chargement du bitstream. Par exemple, définir la valeur à `0` pour indiquer un bitstream complet.
> Plus d'informations sur les types de bitstreams fournies par le [support xilinx](https://support.xilinx.com/s/article/63419?language=en_US)

- **firmware** : Ce fichier est utilisé pour spécifier le nom du bitstream à charger. Le bitstream doit être placé dans le répertoire `/lib/firmware`.

!!! info "Activation du FPGA Manager"
    Avant de pouvoir utiliser FPGA Manager, vous devez vous assurer qu'il est activé dans votre configuration PetaLinux. Suivez les étapes détaillées dans le guide [PetaLinux Tools Documentation: Reference Guide (UG1144)](https://docs.amd.com/r/en-US/ug1144-petalinux-tools-reference-guide/FPGA-Manager-Configuration-and-Usage-for-Zynq-7000-Devices-and-Zynq-UltraScale-MPSoC) pour activer FPGA Manager pour les dispositifs Zynq-7000 et Zynq UltraScale MPSoC.

### Mise à Jour du FPGA depuis Linux

Suivez les étapes ci-dessous pour mettre à jour le FPGA en utilisant FPGA Manager. ([source](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841645/Solution+Zynq+PL+Programming+With+FPGA+Manager))

=== "Sans DTO"

    1. Connectez-vous en tant qu'utilisateur root :

        ```bash
        sudo -i
        ```

    2. Définissez les flags pour le Bitstream complet :

        ```bash
        echo 0 > /sys/class/fpga_manager/fpga0/flags
        ```

    3. Chargez le Bitstream dans le PL :

        ```bash
        mkdir -p /lib/firmware
        cp project.bit.bin /lib/firmware/
        echo project.bit.bin > /sys/class/fpga_manager/fpga0/firmware
        ```

    4. Félicitations, le FPGA a été mis à jour avec succès !

=== "Avec DTO"

    1. **Préparation du Device Tree Overlay (DTO)**

        Assurez-vous que votre DTO est correctement configuré pour votre projet. La configuration et compilation d'un DTO est détaillée dans la section [Les Devices Tree](device_tree.md).

        Vous devez avoir un fichier `.dtbo` prêt à être utilisé pour configurer le FPGA.

    2. **Chargement du DTO et Programmation du FPGA**

        1. Connectez-vous en tant qu'utilisateur root :

            ```bash
            sudo -i
            ```

        2. Définissez les flags pour le Bitstream complet :

            ```bash
            echo 0 > /sys/class/fpga_manager/fpga0/flags
            ```

        3. Copiez le Bitstream et le DTBO dans le répertoire `/lib/firmware` :

            ```bash
            mkdir -p /lib/firmware
            cp project.bit.bin /lib/firmware/project.bit.bin
            cp pl.dtbo /lib/firmware/
            ```

        4. Appliquez le DTBO :
            
            ```bash
            mkdir /configfs
            mount -t configfs configfs /configfs
            cd /configfs/device-tree/overlays/
            mkdir full
            echo -n "pl.dtbo" > full/path
            ```

            !!! note "Remarque"
                Lorsque vous appliquez un DTBO, le FPGA est automatiquement programmé avec le bitstream spécifié dans le DTO.

        5. Félicitations, votre FPGA a été mis à jour avec succès en utilisant un DTO !
        
        !!! tip "Suppression du DTBO"
            Pour supprimer le DTBO :

            ```bash
            cd /configfs/device-tree/overlays/
            rmdir full
            ```
