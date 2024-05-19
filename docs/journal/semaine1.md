### Mardi 14/05

RDV initial avec prof déplacé, recherche et documentation initiale, trois pistes possibles : petalinux, yocto, manuel. On a majoritairement fait du manuel cette journée-là à build
manuellement les projets git xilinx, mauvaise idée + certains éléments ne correspondent pas à notre zybo mais au zc702 (vu seulement la journée d'après)...

### Mercredi 15/05

RDV avec le prof, but : distro linux classique sur la zybo + programmer dynamiquement le fpga, tout ça sur carte sd pour le moment. Petalinux OK (heureusement). Prise en main de l'outil initial
puis on remarque qu'on utilisait des éléments comme le Device Tree de la ZC702... Donc direction le site digilent zybo z7-20 pour des démos -> aucune à jour pour petalinux 2023.2 (dernière version 2022.1).
DONC téléchargement du repo github de la démo Petalinux 2022.1 afin de choper le block diagram et mettre à jour les blocs IP sur vivado (fait auto par vivado d'ailleurs). Export du XSA correspondant
à la démo avec ses quelques fonctionnalités.

### Jeudi 16/05

Import du xsa dans petalinux puis début de config -> activer carte sd, formater bien la carte sd, etc. Build réalisé aussi et téléchargement du rootfs puis transfert du tout sur la carte sd.
Test sur la zybo et ... échec. FSBL/U-Boot fonctionnent nickel mais le kernel n'arrive pas à boot le rootfs
et reste planté sur "bootconsole cdns0 disabled"
Possible moyen de résoudre le pb trouvé : ro est passé dans les command line parameters du kernel à cause d'un bug de petalinux 2023.2

### Vendredi 17/05

rw rajouté dans la config afin que le rootfs soit bien pris en read-write pour fix le bug d'avant mais le problème du bootconsole est toujours présent
Activation des options debug afin d'y voir plus clair... Pas grand chose de trouvé...
Jusqu'à qu'on remarque que le FPGA Manager n'est PAS activé, une fois activé tout marche !
Visiblement deux trois petits pb à résoudre dans le lancement de debian mais on arrive bien à un terminal et on peut commencer à s'amuser