# Device Tree et Device Tree Overlay

## Introduction

Le Device Tree permet de maintenir le noyau Linux générique, sans avoir besoin d'intégrer des détails matériels spécifiques dans le noyau lui-même. Cette flexibilité permet de porter le même noyau sur différentes plateformes matérielles en modifiant simplement le Device Tree. Cette caractéristique est cruciale pour le développement et la maintenance de systèmes embarqués.

### Device Tree

Les Device Trees permettent de visualiser les différents périphériques, composants et registres accessibles par le processeur sous forme d'une structure arborescente. Cette approche organise les informations de manière hiérarchique, facilitant ainsi l'identification et l'accès aux composants spécifiques du système.

La syntaxe utilisée pour définir les Device Trees est la suivante ([elinux.org](https://elinux.org/Device_Tree_Usage)) :

!!! example "Exemple de Device Tree"
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

Les Device Trees doivent être compilés pour être transformés en un format binaire utilisable par le système. Cette étape de compilation est effectuée à l'aide de l'outil Device Tree Compiler (DTC). Le résultat de cette compilation est un Device Tree Blob (DTB) sous l'extension `.dtb`.

Plus de détails concernant les Device Trees sont disponibles sur [Device Tree 101](https://bootlin.com/pub/conferences/2021/webinar/petazzoni-device-tree-101/petazzoni-device-tree-101.pdf), une présentation de [Bootlin](https://bootlin.com/).

!!! note "Séparation des fichiers"
    Il est souvent nécessaire de séparer un Device Tree en plusieurs fichiers. Cette séparation se fait en utilisant les formats suivants :

    - `.dts` (Device Tree Source) : fichier principal du Device Tree, pouvant inclure d'autres fichiers.
    - `.dtsi` (Device Tree Include) : fichiers à inclure dans le fichier principal .dts via des directives #include.

    L'assemblage de ces fichiers séparés est réalisé en utilisant le préprocesseur GCC, permettant ainsi de combiner les différentes parties en un seul fichier cohérent (fichier `.dts` final sans #include).


### Device Tree Overlay

Les Device Trees décrivent les composants matériels pour que le système d'exploitation puisse les reconnaître et les utiliser. Cependant, que faire si l'on veut ajouter un composant alors qu'un Device Tree est déjà en cours d'utilisation et qu'il est impossible d'éteindre le système ? 

La solution se trouve dans les Device Tree Overlays (DTO). Ces morceaux de Device Tree permettent de rajouter, modifier ou même supprimer des nœuds de l'arbre, modifiant ainsi le comportement du système à la volée.

Les fichiers de DTO sont généralement au format `.dtso` et `.dtbo` pour la version compilée. Notez qu'il est aussi possible de voir des overlays en `.dtsi`. Ne vous fiez donc pas uniquement au format de fichier. La principale manière de les différencier se trouve dans leur syntaxe et la mention de `/plugin/` au début du fichier.

Deux syntaxes existent : une ancienne et une nouvelle.

!!! example "Exemple de Device Tree Overlay"
    === "Ancienne"
        ```dts
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
    === "Nouvelle"
        ```dts
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

    Ces deux exemples sont fonctionnellement équivalents. La nouvelle syntaxe est souvent préférée pour sa simplicité et sa lisibilité.

### Exemple DTO pour un encodeur

Le Device Tree est généré automatiquement par PetaLinux à partir d'un fichier XSA. Notre travail s'est principalement concentré sur la création de l'overlay. 

L'overlay que nous avons utilisé est écrit avec l'ancienne syntaxe et permet de réaliser plusieurs actions importantes :

!!! abstract "Overlay utilisé pour la reprogrammation de l'encodeur"
    ```dts
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

Dans le cas des FPGA Zynq, le `fragment@0` de l'overlay modifie la propriété `firmware-name` de la région FPGA/PL, spécifiant ainsi le bitstream à utiliser pour la reprogrammation. 

Le `fragment@1` ajoute un nouveau composant au bus AMBA/AXI à l'adresse mémoire `0x40000000`. Il spécifie également la clock utilisée par le composant et son nom via `clock-names` et `clocks`, le driver à utiliser pour communiquer avec l'appareil via la propriété `compatible`, et enfin l'adresse mémoire attribuée au composant ainsi que la taille de cette zone via `reg`.

Ces ajouts permettent au kernel d'accéder aux registres de notre bloc IP avec le driver UIO, facilitant ainsi l'interaction avec notre block IP.

??? tip "Explication détaillée"
    - **fragment@0** :
        - **target** : Pointe vers la région FPGA/PL.
        - **__overlay__** : 
            - **firmware-name** : Spécifie le nom du bitstream (`enc_wrapper.bit.bin`) à charger pour la reprogrammation du FPGA.

    - **fragment@1** :
        - **target** : Pointe vers le bus AMBA/AXI.
        - **__overlay__** :
            - **PetaENC_0** : Nom du composant ajouté.
            - **clock-names** : Nom de la clock utilisée (`s_axi_aclk`).
            - **clocks** : Référence à la clock configurée (15^ème^ entrée dans `clkc`).
            - **compatible** : Spécifie le driver (`generic-uio`) à utiliser pour communiquer avec le composant.
            - **reg** : Spécifie l'adresse mémoire (`0x40000000`) et la taille de la zone mémoire allouée (`0x1000`).

    En suivant cette structure, le kernel peut accéder aux registres de l'IP personnalisé en utilisant le driver UIO.

---

## Génération de Device Tree

### Contexte

La génération du Device Tree est généralement prises en charge de manière automatique par des outils tels que PetaLinux, qui s'appuient sur les fichiers XSA pour produire ces éléments. Toutefois, dans certaines situations, il peut être intéressant de générer soi-même le Device Tree.

### Prérequis

- Vivado et Vitis installés
- Fichier XSA généré

### Procédure

1. Lancer XSCT

    Ouvrez un terminal et lancez XSCT :

    ```
    xsct
    ```

    !!! note "Note"
        Si vous avez des problèmes pour lancer XSCT, lancer un terminal depuis vitis et exécuter la commande `xsct`.

2. Charger le fichier XSA

    Utilisez la commande suivante pour charger votre fichier XSA :

    ```tcl
    hsi open_hw_design <path_to_your_xsa_file>
    ```

3. Specifier le DTG repository


    ```
    git clone https://github.com/Xilinx/device-tree-xlnx
    cd device-tree-xlnx
    git checkout <xilinx_rel_v20XX.X>
    ```

    Spécifiez le répertoire DTG (Device Tree Generator) :
    ```tcl
    hsi set_repo_path <path to device-tree-xlnx repository>
    ```

4. SW design

    Utilisez la commande suivante pour obtenir la liste des processeurs disponibles :

    ```tcl
    set procs [hsi get_cells -hier -filter {IP_TYPE==PROCESSOR}]
    ```

    Choisissez le processeur que vous souhaitez utiliser pour générer le Device Tree :
    ```tcl
    hsi create_sw_design device-tree -os device_tree -proc <proc_name>
    ```

5. Générer le Device Tree

    ```tcl
    hsi generate_target -dir <path to output file>
    ```

6. Fermer XSCT

    ```tcl
    hsi close_hw_design [hsi current_hw_design]
    exit
    ```

En suivant cette procédure, vous serez en mesure de générer . Pour compiler le Device Tree généré, référez-vous à la [documentation Xilinx officielle](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842279/Build+Device+Tree+Blob#BuildDeviceTreeBlob-CompilingDevicetreeSources).

---

## Compilation d'un Device Tree Overlay

### Contexte

Le Device Tree Overlay (DTO) pour le Zynq ne se génère pas automatiquement; il faut donc l'écrire soi-même. Toutefois, la compilation du DTO est relativement simple grâce aux outils appropriés. Voici les étapes détaillées pour compiler un Device Tree Overlay.

### Prérequis

Pour compiler un Device Tree Overlay, vous aurez besoin du Device Tree Compiler (DTC). Ce dernier est fourni avec Vitis ou peut être téléchargé depuis le repository officiel [ici](https://git.kernel.org/pub/scm/utils/dtc/dtc.git).

### Procédure

```
dtc -O dtb -o FICHIER.dtbo -b 0 -@ FICHIER.dtso
```