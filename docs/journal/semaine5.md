### Lundi 10/06

Décision prise de debug le bloc IP custom en passant par des simulations : abandon de l'utilisation du mode buffer en faveur du mode inout (recommendation officielle de Xilinx de toute façon).

### Mardi 11/06

Debug du bloc IP custom : registres tristate du Pmod Bridge non-configurés en input donc ajout d'un bloc Constant pour le faire, encore quelques fix niveau switch buffer/inout et sur numeric_std pour les conversions UNSIGNED en STD_LOGIC_VECTOR.

Développement d'une appli de test plus pratique côté Linux, check direct d'une adresse d'un device UIO avec polling toutes les secondes en option.

**Encodeur fonctionnel et continuité des opérations niveau Linux assurée.**

### Mercredi 12/06

Discussion projet, avancées mineures

### Jeudi 13/06

### Vendredi 14/06