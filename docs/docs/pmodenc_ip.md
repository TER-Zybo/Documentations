# Digilent PmodENC - Bloc IP personnalisé

## Introduction

Le Digilent PmodENC est un module d'entrée rotatif qui permet aux utilisateurs d'ajouter une interface de contrôle rotative à leurs projets FPGA. Ce module inclut un encodeur rotatif, un bouton-poussoir intégré et un switch. Plus d'informations sur ce composant peuvent être trouvées sur la [page officielle du PmodENC](https://digilent.com/reference/pmod/pmodenc/start).

Digilent propose un bloc IP officiel pour le PmodENC, mais il ne correspond pas à nos besoins car son utilisation nécessite de coder le compteur 4 bits et l'antirebond du côté PS au lieu du côté PL. Nous avons donc créé un simple bloc IP incluant un compteur 4 bits en module RTL codé en VHDL.

## Conception

Le bloc IP PetaENC est constitué de :

- Un bloc AXI GPIO, contenant des registres accessibles par bus AXI.
- Le module RTL du compteur, comportant également un antirebond, le tout en VHDL.
- Un bloc Pmod Bridge, permettant d'interfacer le Pmod avec le module.


## Utilisation

Le bloc IP PetaENC est à relier à un AXI Interconnect maître et aux broches du connecteur Pmod. Il est recommandé de passer par un fichier de contraintes XDC pour connecter le port Pmod à des broches externes.

### Ajout du Bloc IP à Vivado

1. **Cloner le Répertoire IP** :
    Tout d'abord, clonez le répertoire sur votre machine locale :

    ```sh
    git clone https://github.com/TER-Zybo/PetaENC_IP
    ```

2. **Ouvrir Vivado** :
    Lancez Xilinx Vivado Design Suite sur votre ordinateur.

3. **Ajouter le Répertoire IP** :
    - Dans Vivado, allez dans `Tools > Project Settings`.
    - Naviguez jusqu'à `IP > Repository`.
    - Cliquez sur le bouton `+` pour ajouter un nouveau répertoire IP.
    - Parcourez l'emplacement du répertoire cloné et sélectionnez le répertoire.
    - Cliquez sur `OK` pour ajouter le répertoire.

4. **Ajouter l'IP à Votre Projet** :
    - Une fois le répertoire ajouté, vous pouvez ajouter le bloc IP PetaENC à votre projet depuis le catalogue IP.

### Exemple de Fichier de Contraintes (XDC)

Voici un exemple de la manière dont vous pouvez mapper le port Pmod à des broches externes en utilisant un fichier XDC :

```xdc
set_property PACKAGE_PIN <numéro_de_broche> [get_ports {<nom_du_port>}]
set_property IOSTANDARD LVCMOS33 [get_ports {<nom_du_port>}]
```

Remplacez `<numéro_de_broche>` et `<nom_du_port>` par les valeurs appropriées pour votre conception.

!!! example "Configuration recommandée pour le JE"
    ```xdc
    set_property -dict { PACKAGE_PIN V12   IOSTANDARD LVCMOS33 } [get_ports { Pmod_out_0_pin1_io }]; #IO_L4P_T0_34 Sch=je[1]						 
    set_property -dict { PACKAGE_PIN W16   IOSTANDARD LVCMOS33 } [get_ports { Pmod_out_0_pin2_io }]; #IO_L18N_T2_34 Sch=je[2]                     
    set_property -dict { PACKAGE_PIN J15   IOSTANDARD LVCMOS33 } [get_ports { Pmod_out_0_pin3_io }]; #IO_25_35 Sch=je[3]                          
    set_property -dict { PACKAGE_PIN H15   IOSTANDARD LVCMOS33 } [get_ports { Pmod_out_0_pin4_io }]; #IO_L19P_T3_35 Sch=je[4]                     
    set_property -dict { PACKAGE_PIN V13   IOSTANDARD LVCMOS33 } [get_ports { Pmod_out_0_pin7_io }]; #IO_L3N_T0_DQS_34 Sch=je[7]                  
    set_property -dict { PACKAGE_PIN U17   IOSTANDARD LVCMOS33 } [get_ports { Pmod_out_0_pin8_io }]; #IO_L9N_T1_DQS_34 Sch=je[8]                  
    set_property -dict { PACKAGE_PIN T17   IOSTANDARD LVCMOS33 } [get_ports { Pmod_out_0_pin9_io }]; #IO_L20P_T3_34 Sch=je[9]                     
    set_property -dict { PACKAGE_PIN Y17   IOSTANDARD LVCMOS33 } [get_ports { Pmod_out_0_pin10_io }]; #IO_L7N_T1_34 Sch=je[10]    
    ```
    Merci de remplacer `Pmod_out_0` par le nom du port de sortie du bloc IP PetaENC.

## Références

- [Pmod ENC](https://digilent.com/reference/pmod/pmodenc/start)
- [Répertoire IP PetaENC](https://github.com/TER-Zybo/PetaENC_IP)
- [Bibliothèque Vivado de Digilent 2019.1](https://github.com/Digilent/vivado-library/)