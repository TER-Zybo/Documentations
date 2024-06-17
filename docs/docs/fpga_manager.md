# Programmation FPGA depuis Linux avec FPGA Manager

## Introduction

Les FPGA (Field-Programmable Gate Arrays) sont des circuits intégrés reprogrammables qui permettent une grande flexibilité pour diverses applications. Pour programmer ces dispositifs, des fichiers spécifiques sont utilisés : les fichiers BIT et BIN. Ce guide vous expliquera comment générer un fichier BIN à partir d'un fichier BIT et l'utiliser pour programmer votre FPGA, en utilisant l'outil FPGA Manager sous Linux.

## FPGA Manager : Fonctionnement et Utilité

Le gestionnaire FPGA (FPGA Manager) est une interface du noyau Linux permettant de charger et gérer les bitstreams des FPGA. Il permet une interaction simplifiée avec le FPGA, en exposant des fichiers virtuels à travers le système de fichiers Linux.

### Principaux Composants de FPGA Manager

- **flags** : Ce fichier permet de définir des options pour le chargement du bitstream. Par exemple, définir à `0` pour un bitstream complet.
- **firmware** : Ce fichier est utilisé pour spécifier le nom du bitstream à charger. Le bitstream doit être placé dans `/lib/firmware`.

## Mise à jour du FPGA depuis Linux

### Activer FPGA Manager

Avant de pouvoir utiliser FPGA Manager, vous devez vous assurer qu'il est activé dans votre configuration PetaLinux. Suivez les étapes détaillées dans le guide [PetaLinux Tools Documentation: Reference Guide (UG1144)](https://docs.amd.com/r/en-US/ug1144-petalinux-tools-reference-guide/FPGA-Manager-Configuration-and-Usage-for-Zynq-7000-Devices-and-Zynq-UltraScale-MPSoC) pour activer FPGA Manager pour les dispositifs Zynq-7000 et Zynq UltraScale MPSoC.

### Préparation du Fichier BIN

Avant de pouvoir charger le bitstream dans le FPGA, vous devez générer un fichier BIN à partir du fichier BIT. Suivez les étapes ci-dessous pour générer un fichier BIN à partir d'un fichier BIT.

#### Fichiers BIT et BIN : Quelles différences ?

| Caractéristique      | Fichiers BIT (Bitstream)                                              | Fichiers BIN                                                |
|----------------------|-----------------------------------------------------------|-------------------------------------------------------------|
| **Contenu**          | Ce bitstream inclut les informations nécessaires pour programmer les ressources logiques et les interconnexions du FPGA. | Versions binaires du bitstream de configuration. Compatibles avec les outils de gestion FPGA sous Linux. |
| **Génération**       | Générés par des outils de synthèse et de placement/routage comme `Vivado`. | Générés à partir des fichiers BIT en utilisant l'outil `bootgen`.

#### Comment générer un fichier BIN à partir d'un fichier BIT ?

!!! warning "NE PAS UTILISER VIVADO !"
    Vivado génère des fichiers BIN qui ne sont pas compatibles avec le chargement du FPGA depuis Linux.
    
    ```plaintext	
    Paramètres -> Bitstream -> -bin_file : activer
    ```

    Le fichier sera généré dans `proj_name/proj_name.runs/impl_1` ne sera pas compatible avec le chargement du FPGA depuis Linux.
    
    l'utilisation de `bootgen` est nécessaire pour générer un fichier BIN compatible.

1. Créez un fichier de configuration BIF nommé avec le contenu suivant :

    ```plaintext title="project.bif"
    all:
    {
        project.bit /* Nom du fichier Bitstream */
    }
    ```

2. Utilisez la commande `bootgen` pour générer le fichier BIN :

    ```bash
    bootgen -image project.bif -arch zynq -process_bitstream bin
    ```

### Utilisation de FPGA Manager

1. Connectez-vous en tant qu'utilisateur root :

    ```bash
    sudo -i
    ```

2. Définissez les flags pour le Full Bitstream :

    ```bash
    echo 0 > /sys/class/fpga_manager/fpga0/flags
    ```

3. Chargez le Bitstream dans le PL :

     ```bash
     mkdir -p /lib/firmware
     cp project.bit.bin /lib/firmware/
     echo project.bit.bin > /sys/class/fpga_manager/fpga0/firmware
     ```

4. Félicitations, vous avez terminé !

!!! info "Programmation FPGA avec DTO"
    Merci de vous référer à la section correspondante disponible [ici](device_tree.md).
