# Problèmes rencontrés
---

## 🗓️ Semaine 1

### 🔴 Problèmes
- Énorme latence lors de l’écriture sur les terminaux.  
  - Les lettres ne correspondaient pas → beaucoup d’erreurs de saisie.

### 🟢 Solutions
- Utilisation d’un PC séparé avec un câble bien tendu.  
- Utilisation d’un ordinateur portable sous Windows XP, plus compatible avec le switch via le port console.  
- Connexions SSH à distance depuis mon ordinateur.  

---

## 🗓️ Semaine 2

### 🔴 Problèmes
- Problème de liaison **Trunk** entre le switch cœur et le switch de la baie.  
- Interconnexion entre le switch et le routeur (VLAN interco).  
- **ACL** : mauvaise configuration.  
- **Mauvaise configuration DNS** :  
  - Création d’un nouveau DNS.  
  - Suppression du DNS initial (lié au compte Administrateur).  

### 🟢 Solutions
- Arrêter d’essayer d’établir une liaison entre les deux appareils.  
- Ne pas supprimer le DNS principal et bien le configurer dès le départ.  
- ⚠️ Cloner la VM pour éviter de devoir refaire toute l’installation.  
- Créer le VLAN d’interconnexion et **le mettre en mode Access** sur une interface pour l’activer. Sinon, il reste inactif et ne peut pas être relié à une liaison Trunk.  
- Refaire les ACL correctement en autorisant les bons réseaux.  

---

## 🗓️ Semaine 3

### 🔴 Problèmes
- Aucun gros problème, à part une perte de temps en salle serveur due à une mauvaise configuration du switch de la baie des serveurs.

### 🟢 Solutions
- Appel du professeur pour modifier la configuration du switch.  
✅ **Test effectué** :  
VLAN actif après configuration.

---

## 🗓️ Semaine 4

### 🔴 Problèmes
- 1️⃣ Problème d’installation du RAID 5 sur le serveur, car nous n’avions aucun accès à HPSSA ni à l’ILO.  
  - L’ILO était corrompue, affichant la version *255.255*, alors qu’elle aurait dû être en version 4 ou 5, ce qui posait problème.  
- 2️⃣ Reconfiguration de l’infrastructure avec l’ajout du pare-feu.

### 🧪 Tests effectués
- Coupure de l’alimentation du serveur afin de tenter de restaurer les paramètres de l’ILO.  
- Flash : mise à jour du firmware sur la puce qui gère l’ILO.

### 🟢 Solutions
- 1️⃣, 2️⃣ Aucune cette semaine.

---

## 🗓️ Semaine 5

### 🔴 Problèmes
- 1️⃣ Le VLAN client n’avait plus accès à Internet depuis l’ajout du pare-feu.

### 🟢 Solutions
- 1️⃣ Modifications des configurations effectuées la semaine 4.

---

## 🗓️ Semaine 6

### 🔴 Problèmes
- 1️⃣ Problème de démarrage d’OPNsense sur le serveur Nutanix (redémarrage à zéro à chaque démarrage de la machine virtuelle).  
- 2️⃣ Impossible de contacter la salle des serveurs depuis notre poste de travail.

### 🟢 Solutions
- 2️⃣ La modification apportée sur le switch la semaine passée manquait une commande pour que le LACP fonctionne correctement (le port-channel et les interfaces des deux ports doivent être identiques, ce qui n’était pas le cas).  
