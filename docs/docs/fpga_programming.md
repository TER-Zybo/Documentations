# Programmation FPGA depuis Linux

## Introduction

Les fichiers BIN et BIT sont des formats de fichier utilisés pour programmer les FPGA (Field-Programmable Gate Arrays). Ce guide détaille comment générer un fichier BIN à partir d'un fichier BIT et l'utiliser pour programmer votre FPGA.

## Fichiers BIT et BIN : Quelles différences ?

| Caractéristique      | Fichiers BIT (Bitstream)                                              | Fichiers BIN                                                |
|----------------------|-----------------------------------------------------------|-------------------------------------------------------------|
| **Contenu**          | Ce bitstream inclut les informations nécessaires pour programmer les ressources logiques et les interconnexions du FPGA. | Versions binaires du bitstream de configuration. Compatibles avec les outils de gestion FPGA sous Linux. |
| **Génération**       | Générés par des outils de synthèse et de placement/routage comme `Vivado`. | Générés à partir des fichiers BIT en utilisant l'outil `bootgen`.

## Comment générer un fichier BIN

!!! warning "NE PAS UTILISER VIVADO !"
    Vivado génère des fichiers BIN qui ne sont pas compatibles avec le chargement du FPGA depuis Linux.
    
    ```plaintext	
    Paramètres -> Bitstream -> -bin_file : activer
    ```

    Le fichier sera généré dans `proj_name/proj_name.runs/impl_1` ne sera pas compatible avec le chargement du FPGA depuis Linux.
    
    l'utilisation de `bootgen` est nécessaire pour générer un fichier BIN compatible.


1. Créez un fichier de configuration BIF nommé avec le contenu suivant :

    ```plaintext
    all:
    {
        project.bit /* Nom du fichier Bitstream */
    }
    ```

2. Utilisez la commande `bootgen` pour générer le fichier BIN :

    ```bash
    bootgen -image project.bif -arch zynq -process_bitstream bin
    ```

## Mise à jour du FPGA depuis Linux

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