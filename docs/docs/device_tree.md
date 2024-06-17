# Device Tree 

## Introduction

Le Device Tree permet de maintenir le noyau Linux générique, sans avoir besoin de coder des détails matériels spécifiques dans le noyau lui-même. Cela permet de porter le même noyau sur différentes plateformes matérielles, simplement en modifiant le Device Tree. Cette flexibilité est cruciale pour le développement et la maintenance de systèmes embarqués, étant donné la nature des processeurs ARM et RISC-V souvent utilisés dans le domaine.

## Syntaxe

La syntaxe est la suivante (https://elinux.org/Device_Tree_Usage) :

```
/dts-v1/;

/ {
    node1 {
        a-string-property = "A string";
        a-string-list-property = "first string", "second string";
        // hex is implied in byte arrays. no '0x' prefix is required
        a-byte-data-property = [01 23 34 56];
        child-node1 {
            first-child-property;
            second-child-property = <1>;
            a-string-property = "Hello, world";
        };
        child-node2 {
        };
    };
    node2 {
        an-empty-property;
        a-cell-property = <1 2 3 4>; /* each number (cell) is a uint32 */
        child-node1 {
        };
    };
};
```

Elle permet de visualiser les différents devices, composants et registres qui seront accessibles par le processeur sous forme d'arbre.
Il est possible d'éclater un Device Tree et de le séparer en plusieurs fichiers afin de les combiner plus tard en passant par des "#include". L'assemblage de ces fichiers peut être réalisé en passant par GCC.

En suivant cette logique de séparation, les Device Trees ont plusieurs formats :
- .dts : pour le Device Tree principal, pouvant inclure d'autres fichiers
- .dtsi : pour les Device Tree à inclure dans le principal avec des "#include"

Les Device Trees entiers, donc les .dts sans "#include", doivent passer par une étape de compilation afin d'être transformés en format binaire, dtc (Device Tree Compiler) est l'outil utilisé.
Une fois compilé, un fichier .dtb, pour Device Tree Blob, est créé.

Plus de détails concernant les Device Trees sont disponibles sur cette présentation (https://bootlin.com/pub/conferences/2021/webinar/petazzoni-device-tree-101/petazzoni-device-tree-101.pdf).

## Overlays

Les Device Trees décrivent des composants pour que le système puisse les reconnaître et les utiliser, mais comment faire si l'on veut ajouter un composant alors qu'un Device Tree est déjà créé et utilisé et qu'il est impossible d'éteindre le système ?

La solution se trouve dans les Device Tree Overlays. Ces morceaux de Device Tree permettent de rajouter, modifier ou même supprimer des noeuds de l'arbre, et donc de modifier le comportement du système.

Les fichiers de Device Tree Overlay sont la majorité du temps en format .dtso et .dtbo pour la version compilée. Notez qu'il est possible aussi de voir des overlays en .dtsi, ne vous fiez donc pas uniquement au format de fichier. La principale manière de les différencier se trouve dans leur syntaxe et la mention de "/plugin/" au début.

Deux syntaxes existent, une nouvelle et une ancienne :

!!! note "Ancienne syntaxe"
    ```
    /dts-v1/;
    /plugin/;

    / {

        fragment@0 {
            target = <&fpga_full>;
            __overlay__ {
                firmware-name = "enc_wrapper.bit.bin";
            };
        };

        fragment@1 {
            target = <&amba>;
            __overlay__ {
                PetaENC_0: PetaENC@40000000 {
                    clock-names = "s_axi_aclk";
                    clocks = <&clkc 15>;
                    compatible = "generic-uio";
                    reg = <0x40000000 0x1000>;
                };
            };
        };

    };
    ```

!!! note "Nouvelle syntaxe"
    ```
    /dts-v1/;
    /plugin/;

        &fpga_full {
            firmware-name = "enc_wrapper.bit.bin";
        };

        &amba {
            PetaENC_0: PetaENC@40000000 {
                clock-names = "s_axi_aclk";
                clocks = <&clkc 15>;
                compatible = "generic-uio";
                reg = <0x40000000 0x1000>;
            };
        };
    ```

Ces deux exemples sont fonctionellement équivalents.

Cette documentation décrit les étapes pour générer un Device Tree Source (DTS) à partir d'un fichier XSA en utilisant XSCT (Xilinx Software Command-line Tool).

## Encodeur

Le Device Tree de base est déjà généré par PetaLinux dès lors qu'on lui fournit un fichier XSA, le travail que nous avons réalisé se repose donc surtout sur l'overlay. 

L'overlay utilisé est écrit en utilisant l'ancienne syntaxe et permet plusieurs choses :
!!! note "Overlay utilisé pour la reprog encodeur"
    ```
    /dts-v1/;
    /plugin/;

    / {

        fragment@0 {
            target = <&fpga_full>;
            __overlay__ {
                firmware-name = "enc_wrapper.bit.bin";
            };
        };

        fragment@1 {
            target = <&amba>;
            __overlay__ {
                PetaENC_0: PetaENC@40000000 {
                    clock-names = "s_axi_aclk";
                    clocks = <&clkc 15>;
                    compatible = "generic-uio";
                    reg = <0x40000000 0x1000>;
                };
            };
        };

    };
    ```

Dans le cas des FPGA Zynq, le "fragment@0" de l'overlay modifie la propriété "firmware-name" de la région FPGA/PL, donnant ainsi le nom du bitstream à utiliser pour la reprogrammation. 

Le "fragment@1" quant à lui rajouter un nouveau composant disponible sur le bus AMBA/AXI à l'adresse mémoire 0x40000000. On retrouve la clock utilisée par le composant et son nom avec "clock-names" et "clocks", le driver à utiliser pour communiquer avec l'appareil avec la propriété "compatible" et enfin l'adresse mémoire attribuée au composant avec la taille de cette zone avec "reg".

Ces ajouts permettent au kernel d'accéder aux registres de notre bloc IP avec le driver UIO. 

## Génération de Device Tree

### Contexte

PetaLinux génère et compile automatiquement le Device Tree à partir du XSA, toutefois il peut être intéressant de le faire soi-même. Voici donc la méthode pour générer le Device Tree. 

Pour le compiler, suivez la documentation Xilinx officielle disponible à cette adresse : https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842279/Build+Device+Tree+Blob#BuildDeviceTreeBlob-CompilingDevicetreeSources.

### Prérequis

- Vivado et Vitis installés
- Fichier XSA généré

### Étapes

#### 1. Lancer XSCT

Ouvrez un terminal et lancez XSCT :

```bash
xsct
```

!!! note "Note"
    Si vous avez des problèmes pour lancer XSCT, lancer un terminal depuis vitis et exécuter la commande `xsct`.

#### 2. Charger le fichier XSA

Utilisez la commande suivante pour charger votre fichier XSA :

```tcl
hsi open_hw_design <path_to_your_xsa_file>
```

#### 3. Specifier le DTG repository


```bash
git clone https://github.com/Xilinx/device-tree-xlnx
cd device-tree-xlnx
git checkout <xilinx_rel_v20XX.X>
```

Spécifiez le répertoire DTG (Device Tree Generator) :
```tcl
hsi set_repo_path <path to device-tree-xlnx repository>
```

#### 4. SW design

Utilisez la commande suivante pour obtenir la liste des processeurs disponibles :

```tcl
set procs [hsi get_cells -hier -filter {IP_TYPE==PROCESSOR}]
```

Choisissez le processeur que vous souhaitez utiliser pour générer le Device Tree :
```tcl
hsi create_sw_design device-tree -os device_tree -proc <proc_name>
```

#### 5. Générer le Device Tree

```tcl
hsi generate_target -dir <path to output file>
```

#### 6. Fermer XSCT

```tcl
hsi close_hw_design [hsi current_hw_design]
exit
```

## Compilation d'un Device Tree Overlay

### Contexte

Un Device Tree Overlay pour le Zynq ne se génère pas automatiquement, il faut donc l'écrire soi-même. Toutefois, la partie compilation est beaucoup plus simple, les étapes sont ci-dessous.

### Prérequis

    - Device Tree Compiler, fourni avec Vitis et sinon disponible en clonant le répertoire git https://git.kernel.org/pub/scm/utils/dtc/dtc.git suivi d'un make

#### Étape de compilation

```
dtc -O dtb -o FICHIER.dtbo -b 0 -@ FICHIER.dtso
```

## Références

Pour plus de détails, consultez la [documentation Xilinx](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842279/Build+Device+Tree+Blob).