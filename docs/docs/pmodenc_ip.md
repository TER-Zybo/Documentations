# PetaENC - Bloc IP personnalisé

Introduction

Digilent propose un bloc IP officiel pour le PmodENC, or il ne correspond pas à nos besoins étant donné que son utilisation revient à coder le compteur 4 bits et l'antirebond du côté PS et non du côté PL. Nous avons donc créé un simple bloc IP incluant un compteur 4 bits en module RTL codé en VHDL.

Conception

Le bloc IP PetaENC est constitué de :

- Un bloc AXI GPIO, contenant des registres accessibles par bus AXI
- Le module RTL du compteur, comportant aussi un antirebond, le tout en VHDL
- Un bloc Pmod Bridge, permettant d'interfacer le Pmod avec le module

METTRE IMAGE

Utilisation

Le bloc IP PetaENC est à relier à un AXI Interconnect maître et aux pins du connecteur Pmod, il est recommandé de passer par un fichier de contraintes XDC pour ce dernier en mettant le port en externe.