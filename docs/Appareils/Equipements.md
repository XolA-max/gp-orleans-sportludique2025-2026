# Présentation des équipements réseau

## 🌐 Switch CORE

### **Stack** : 
- Empilement de switches (tolérance aux pannes + plus de ports).
### **SSH** : 
- Accès distant via VLAN de management.
### **LACP** : 
- Agrégation de liens (Fa1/0/23 et Fa2/0/23).
### **Mode Trunk** : 
- Fa1/0/23-24 et Fa2/0/23-24 → tous les VLANs.
### **Mode Access** : 
- Fa1/0/1-4 → un seul VLAN.
### **VLANs** : 
- Management
- Clients
- Serveurs
- Interco
### **Routage** : 
- Activation du routage.
### **Passerelle par default** : 
- Route par default sur IP de la passerelle virtuel
### **Relai DHCP** : 
- Contacter le serveur AD pour obtenir une adresse ip en DHCP
### **Liaisons** :
  - Fa1/0/1 à Fa1/0/4 → ordinateurs clients  
  - Fa1/0/23 & Fa2/0/23 → switch B4  
  - Fa1/0/24 & Fa2/0/24 → routeur Fibre + ADSL  

---

## 🌐 Switch Salle Serveur

### **SSH** : 
- Accès distant (3 comptes utilisateurs avec droits complets).
### **LACP** : 
- Agrégation de liens (Fa1/0/1 et Fa1/0/2).
### **Mode Trunk** : 
- Fa1/0/1-2 → tous les VLANs 
- Fa1/0/21-23 → Vlan 241
### **Mode Access** : 
- Fa1/0/24 → Vlan Mana.
### **VLANs** : 
- Management
- Serveurs.
### **Liaisons** :
  - Fa1/0/1 & Fa1/0/2 → vers les switchs salle serveur  
  - Fa1/0/23 & Fa1/0/24 → vers switch serveur  

---

## 📡 Routeur Fibre

### **SSH** : 
- Accès distant (3 comptes utilisateurs avec droits complets).
### **VLANs** : 
- Management, Interco.
### **Routage statique** :
  - Utiliser l’interface Gi0/0.interco pour accéder aux VLAN Serveurs et Clients.
### **ACL/NAT** :
  - Autoriser le VLAN Interco à sortir du routeur pour accéder à Internet
### **HA (VRRP)** :
  - Création d’une passerelle virtuelle permettant d’avoir plusieurs passerelles : une active et une inactive. Lorsque la passerelle active tombe en panne, le basculement se fait automatiquement vers l’autre routeur.
### **Interfaces** :
  - Gi0/0 (virtuel) : VLAN Interco(nat inside) + VLAN Management (mode access).
  - Gi0/1 : VLAN 200 (mode access,nat outside).

---

## 📡 Routeur ADSL

### **SSH** : 
  - Accès distant (3 comptes utilisateurs avec droits complets).
### **VLANs** : 
  - Management 
  - Interco.
### **Routage statique** :
  - Utiliser l’interface Gi0/0.interco pour accéder aux VLAN Serveurs et Clients.
### **ACL/NAT** :
  - Autoriser le VLAN Interco à sortir du routeur pour accéder à Internet
### **HA** :
  - Création d’une passerelle virtuelle permettant d’avoir plusieurs passerelles : une active et une inactive. Lorsque la passerelle active tombe en panne, le basculement se fait automatiquement vers l’autre routeur.
### **Interfaces** :
  - Gi0/0 (virtuel) : VLAN Interco(nat inside) + VLAN Management (mode access).
  - Gi0/1 : VLAN 100 (mode access,nat outside).

---

## 🚫 PareFeu physique stormshield

---

### Accès à l’interface

- L’administration se fait via l’interface web.  
- Adresse IP par défaut ou configurée pour le management : `192.168.140.45`  
- Utiliser un navigateur moderne et se connecter avec le compte administrateur.  

---

#### Règles de filtrage

Actuellement, aucune règle de filtrage n’a été mise en place afin de vérifier le bon fonctionnement de notre réseau.

---

#### Interfaces

- Gi0/0 Lan → Vlan 245
- GI0/1 WAn → Vlan 248

---
#### Routes

- Les routes configurées sur le pare-feu permettent notamment de redescendre vers le réseau **172.28.96.0/19**.  
- Une **route par défaut** est définie vers un **HSRP** (Hot Standby Router Protocol) qui réunit les deux routeurs.  

---

## 🚫 PareFeu virtuel

### Accès à l’interface

- L’administration se fait via l’interface web.  
- Adresse IP par défaut ou configurée pour le management : `192.168.140.75`  
- Utiliser un navigateur moderne et se connecter avec le compte administrateur.  

---

#### Règles de filtrage

Actuellement, aucune règle de filtrage n’a été mise en place afin de vérifier le bon fonctionnement de notre réseau.

---

#### Routes

- Les routes configurées sur le pare-feu permettent notamment de redescendre vers le réseau **172.28.96.0/19**.  
- Une **route par défaut** est définie vers le switch coeur.  

---

#### Interfaces

- Gi0/0 Lan → Vlan 245
- GI0/1 WAn → Vlan 249

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

## 🔷 Serveur Proxmox

- **Installation RAID 5 avec LVM**
  - Création des partitions avec `fdisk` ou `parted`.
  - Configuration du RAID 5 via `mdadm`
  - Création du volume LVM

- **Installation de Proxmox**
  - Création d’une clé USB bootable avec `Rufus` ou `dd`
  - Démarrage du serveur sur la clé USB et installation de l’OS Proxmox.

- **Configuration de Proxmox**
  - Configuration du VLAN de management dans l’interface réseau :
  - Redémarrage du service réseau :
