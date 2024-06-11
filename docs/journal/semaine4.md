### Lundi 03/06

Discussion à propos de la direction à prendre et des éléments à développer (overlay, tests drivers).

Premiers essais avec Vitis, incluant la création d'une plateforme et d'un projet en C. Recherche sur l'installation du driver Pmod_ENC et tentative de compilation du device tree.

### Mardi 04/06

Nous avons remarqué que les drivers Digilent fournis ne sont pas des LKM (Linux Kernel Modules), ce sont des drivers baremetal. Décision prise sur le développement d'un bloc IP custom afin de bien montrer le reprogrammation PL avec le FPGA manager et de faire un driver custom LKM.

Développement bloc IP custom, reprise du VHDL réalisé plus tôt afin de le mettre en module RTL dans le bloc avec ajout d'un bloc Xilinx AXI GPIO et Digilent Pmod Bridge. Modification du VHDL pour utiliser numeric_std, plus récent et de plus en plus utilisé désormais.

Documentation sur la génération d'un device tree et de la réalisation d'un projet baremetal en C avec vitis.

### Mercredi 05/06

Réalisation de l'overlay en se basant sur le xsa de base et le xsa avec notre bloc IP en plus.

Abandon du développement d'un driver LKM custom, trop compliqué à faire même si toutefois intéressant. Driver/programme côté userspace (type /dev/mem/ ou UIO) OU utilisation du driver GPIO existant envisagés.

### Jeudi 06/06

Oleg part sur l'UIO, Dwayne sur le driver GPIO. 

Modification de l'overlay pour pointer vers le driver UIO générique (generic-uio) et reprog du FPGA avec le FPGA Manager. 

Toutefois, device UIO introuvable dans "/dev/"...
Tentative de lier generic-uio au driver uio_pdrv_genirq dans les rootargs, ne marche pas.
Eléments manquants dans la config petalinux, on reconfigure mais ne marche toujours pas...
En réalité, nous avons oublié de transférer les drivers fournis par petalinux (qui sont packaged dans son rootfs en tant que modules et non le kernel), donc transfert des modules vers notre rootfs Debian.

Tout cela nous amène bien à la détection du device UIO une fois la reprog FPGA réalisée, mais l'encodeur ne marche malheureusement pas.

### Vendredi 07/06

Tentative de résoudre le problème SANS réaliser de simulation de notre bloc IP custom, aucun résultat.