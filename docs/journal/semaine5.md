### Lundi 10/06

Décision prise de debug le bloc IP custom en passant par des simulations : abandon de l'utilisation du mode buffer en faveur du mode inout (recommendation officielle de Xilinx de toute façon).

### Mardi 11/06

Debug du bloc IP custom : registres tristate du Pmod Bridge non-configurés en input donc ajout d'un bloc Constant pour le faire, encore quelques fix niveau switch buffer/inout et sur numeric_std pour les conversions UNSIGNED en STD_LOGIC_VECTOR.

Développement d'une appli de test plus pratique côté Linux, check direct d'une adresse d'un device UIO avec polling toutes les secondes en option.

**Encodeur fonctionnel et continuité des opérations niveau Linux assurée.**

### Mercredi 12/06

Discussion projet, avancées mineures

### Jeudi 13/06

Nous avons présenté le projet à M. Thieblot et discuté de la continuation du TER. Quelques points restent à traiter :
  - Overlays avec interruption
  - Adresse MAC fixe
  - Finalisation de la documentation

### Vendredi 14/06

Mise à jour des descriptions et des fichiers README sur GitHub pour tous les projets. Ajout d'une branche pour les projets Vivado compatibles avec la version 2023.2, car les projets ont été générés avec la version 2023.1 (version disponible sur les machines de la salle 305).