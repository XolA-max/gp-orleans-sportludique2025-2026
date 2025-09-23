# Présentation des équipements réseau

## 🌐 Switch CORE
- **Stack** : Empilement de switches (tolérance aux pannes + plus de ports).
- **SSH** : Accès distant via VLAN de management.
- **LACP** : Agrégation de liens (Fa1/0/23 et Fa2/0/23).
- **Mode Trunk** : Fa1/0/23-24 et Fa2/0/23-24 → tous les VLANs.
- **Mode Access** : Fa1/0/1-4 → un seul VLAN.
- **VLANs** : Management, Clients, Serveurs, Interco.
- **Routage** : Activation du routage.
- **Passerelle par default** : Route par default sur IP de la passerelle virtuel
- **Relai DHCP** : Contacter le serveur AD pour obtenir une adresse ip en DHCP
- **Liaisons** :
  - Fa1/0/1 à Fa1/0/4 → ordinateurs clients  
  - Fa1/0/23 & Fa2/0/23 → switch B4  
  - Fa1/0/24 & Fa2/0/24 → routeur Fibre + ADSL  

---

## 🌐 Switch Salle Serveur
- **SSH** : Accès distant (3 comptes utilisateurs avec droits complets).
- **LACP** : Agrégation de liens (Fa1/0/1 et Fa1/0/2).
- **Mode Trunk** : Fa1/0/1 et Fa1/0/2 → tous les VLANs.
- **Mode Access** : Fa1/0/23 et Fa1/0/24 → un seul VLAN.
- **VLANs** : Management, Serveurs.
- **Liaisons** :
  - Fa1/0/1 & Fa1/0/2 → vers les switchs salle serveur  
  - Fa1/0/23 & Fa1/0/24 → vers switch serveur  

---

## 📡 Routeur Fibre
- **SSH** : Accès distant (3 comptes utilisateurs avec droits complets).
- **VLANs** : Management, Interco.
- **Routage statique** :
  - Utiliser l’interface Gi0/0.interco pour accéder aux VLAN Serveurs et Clients.
- **ACL/NAT** :
  - Autoriser le VLAN Interco à sortir du routeur pour accéder à Internet
- **HA** :
  - Création d’une passerelle virtuelle permettant d’avoir plusieurs passerelles : une active et une inactive. Lorsque la passerelle active tombe en panne, le basculement se fait automatiquement vers l’autre routeur.
- **Interfaces** :
  - Gi0/0 (virtuel) : VLAN Interco(nat inside) + VLAN Management (mode access).
  - Gi0/1 : VLAN 200 (mode access,nat outside).

---

## 📡 Routeur ADSL
- **SSH** : Accès distant (3 comptes utilisateurs avec droits complets).
- **VLANs** : Management, Interco.
- **Routage statique** :
  - Utiliser l’interface Gi0/0.interco pour accéder aux VLAN Serveurs et Clients.
- **ACL/NAT** :
  - Autoriser le VLAN Interco à sortir du routeur pour accéder à Internet
- **HA** :
  - Création d’une passerelle virtuelle permettant d’avoir plusieurs passerelles : une active et une inactive. Lorsque la passerelle active tombe en panne, le basculement se fait automatiquement vers l’autre routeur.
- **Interfaces** :
  - Gi0/0 (virtuel) : VLAN Interco(nat inside) + VLAN Management (mode access).
  - Gi0/1 : VLAN 100 (mode access,nat outside).

---

## 🚫 PareFeu physique stormshield
 - 
 
---

## 🚫 PareFeu virtuel
 - 

---
## 🔷  Machine virtuel AD
- **Création de la machine viruel** : Création de la machine virtuelle via l’interface web
  - Configuration des ressources nécessaires à la machine virtuelle.
  - Installation de Windows Server 2025.
  - Configurations reseaux : Une interface dans le VLAN Management et une autre dans le VLAN Serveur.
- **AD** :
  - Crétion de l'active directori
- **DHCP** : 
 - Une plage DHCP pour le VLAN Client a été configurée de X.X.X.10 à X.X.X.50.
- **DNS** :
  - Résolution de noms en adresses IP
- **Serveur de fichier** :
  - Fichier partagé entre les comptes clients

---

## 🔷  Serveur Proxmox

