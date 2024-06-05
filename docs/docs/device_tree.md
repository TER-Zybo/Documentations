# Génération d'un Device Tree à partir d'un fichier XSA avec XSCT

## Introduction

Cette documentation décrit les étapes pour générer un Device Tree Source (DTS) à partir d'un fichier XSA en utilisant XSCT (Xilinx Software Command-line Tool).

## Prérequis

- Vivado et Vitis installés
- Fichier XSA généré

## Étapes

### 1. Lancer XSCT

Ouvrez un terminal et lancez XSCT :

```bash
xsct
```

!!! note "Note"
    Si vous avez des problèmes pour lancer XSCT, lancer un terminal depuis vitis et exécuter la commande `xsct`.

### 2. Charger le fichier XSA

Utilisez la commande suivante pour charger votre fichier XSA :

```tcl
hsi open_hw_design <path_to_your_xsa_file>
```

### 3. Specifier le DTG repository

Spécifiez le répertoire DTG (Device Tree Generator) :

```bash
git clone https://github.com/Xilinx/device-tree-xlnx
cd device-tree-xlnx
git checkout <xilinx_rel_v20XX.X>
```

```tcl
hsi set_repo_path <path to device-tree-xlnx repository>
```

### 4. SW design

Utilisez la commande suivante pour obtenir la liste des processeurs disponibles :

```tcl
set procs [hsi get_cells -hier -filter {IP_TYPE==PROCESSOR}]
```

```tcl
hsi create_sw_design device-tree -os device_tree -proc <proc_name>
```

### 5. Générer le Device Tree

```tcl
hsi generate_target -dir <path to output file>
```

### 6. Fermer XSCT

```tcl
hsi close_hw_design [hsi current_hw_design]
exit
```

## Références

Pour plus de détails, consultez la [documentation Xilinx](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842279/Build+Device+Tree+Blob).