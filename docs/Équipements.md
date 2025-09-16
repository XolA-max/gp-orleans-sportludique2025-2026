# Equipements

Étapes effectuées sur chaque équipement:

## Switch CORE

Stack → Empilement de switches permettant d’assurer une tolérance aux pannes ainsi qu’un plus grand nombre de ports disponibles.

SSH → Installation de SSH sur l’équipement afin de pouvoir le manager depuis le VLAN de management.

LACP → La mise en place de LACP permet de faire de l’agrégation de liens, c’est-à-dire d’utiliser deux câbles physiques qui correspondent à un seul lien logique.Sur les interfaces Fa1/0/23 et Fa2/0/23

Mode Trunk → Les interfaces Fa1/0/23-24 et Fa2/0/23-24 sont configurées en mode trunk et permettent de faire passer tous les VLANs autorisés.

Mode Access → Les interfaces Fa 1/0/1-4 sont configurées en mode access et permettent de faire que un VLANs .

Vlan →  Managment -  Clients - Serveurs - Interco

Routage → Activation du routage afin que le switch puisse router les paquets.

Liaisons → 
- Interfaces Fa1/0/1 à Fa1/0/4 : reliées aux ordinateurs clients

- Interfaces Fa1/0/23 et Fa2/0/23 : reliées au switch B4

- Interfaces Fa1/0/24 et Fa2/0/24 : reliées au routeur Fibre et ADSL

## Switch Salle serveur

## Routeur fibre

→Routes

→Interfaces virtuelles pour les Vlans Mana - interco 

→ACL

## Routeur ADSL

→Routes

→Interfaces virtuelles pour les Vlans Mana - interco 

→ACL