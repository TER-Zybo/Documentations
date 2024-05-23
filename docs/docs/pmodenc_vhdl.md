# Projet VHDL : Contrôle de Compteur avec Encodeur Rotatif PMODENC et Affichage LED

Ce projet démontre comment concevoir un système basé sur VHDL en utilisant Xilinx Vivado pour contrôler un compteur avec l'encodeur rotatif PMODENC. La valeur du compteur est affichée sur 4 LEDs de la carte FPGA Zybo Z7-20.

Ce projet s'inspire du guide [Nexys 3 VHDL Example - ISE 14.2](https://digilent.com/reference/_media/reference/pmod/pmodenc/pmodenc_ise_demo_14-2.zip) de [Digilent](https://digilent.com/reference/pmod/pmodenc/start).

## Configuration des ports

### Pmod JE (Pmod standard)

| Signal | Pin |
|--------|-----|
| A      | V12 |
| B      | W16 |
| BTN    | J15 |
| SWT    | H15 |

### Horloge

| Signal | Pin | Description    |
|--------|-----|----------------|
| clk    | K17 | sysclk du Zybo |

Vous pouvez trouver plus d'informations sur les ports du Zybo Z7-20 [ici](https://github.com/Digilent/digilent-xdc/blob/master/Zybo-Z7-Master.xdc).