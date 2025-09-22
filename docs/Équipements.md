# Présentation des équipements réseau

## 🌐 Switch CORE
- **Stack** : Empilement de switches (tolérance aux pannes + plus de ports).
- **SSH** : Accès distant via VLAN de management.
- **LACP** : Agrégation de liens (Fa1/0/23 et Fa2/0/23).
- **Mode Trunk** : Fa1/0/23-24 et Fa2/0/23-24 → tous les VLANs.
- **Mode Access** : Fa1/0/1-4 → un seul VLAN.
- **VLANs** : Management, Clients, Serveurs, Interco.
- **Routage** : Activation du routage.
- **Liaisons** :
  - Fa1/0/1 à Fa1/0/4 → ordinateurs clients  
  - Fa1/0/23 & Fa2/0/23 → switch B4  
  - Fa1/0/24 & Fa2/0/24 → routeur Fibre + ADSL  

---

## 🖥️ Switch Salle Serveur
- **SSH** : Accès distant (3 comptes utilisateurs avec droits complets).
- **LACP** : Agrégation de liens (Fa1/0/1 et Fa1/0/2).
- **Mode Trunk** : Fa1/0/1 et Fa1/0/2 → tous les VLANs.
- **Mode Access** : Fa1/0/23 et Fa1/0/24 → un seul VLAN.
- **VLANs** : Management, Serveurs.
- **Liaisons** :
  - Fa1/0/1 et Fa1/0/2 → vers les switches salle serveur  
  - Fa1/0/23 & Fa1/0/24 → vers switch serveur  

---

## 🚀 Routeur Fibre
- **SSH** : Accès distant (3 comptes utilisateurs avec droits complets).
- **VLANs** : Management, Interco.
- **Interfaces** :
  - Gi0/0 (virtuel) : VLAN Interco + VLAN Management (mode access).
  - Gi0/1 : VLAN 200 (mode access).
- **Fonctions** :
  - Routage statique  
  - Interfaces virtuelles (VLAN Mana + Interco)  
  - ACL (contrôle d’accès)  

---

## 📡 Routeur ADSL
- **Fonctions** :
  - Routage statique  
  - Interfaces virtuelles (VLAN Mana + Interco)  
  - ACL (contrôle d’accès)  
