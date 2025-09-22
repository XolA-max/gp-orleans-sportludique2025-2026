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
- Problème de liaison Trunk entre le switch cœur et le switch de la baie.  
- Interconnexion entre le switch et le routeur (VLAN interco).  
- ACL : mauvaise configuration.  

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

- Appelle du profeseur pour modifier la configuration du switch
✅ **Test effectué** : VLAN actif après configuration.

---

## 🗓️ Semaine 4

### 🔴 Problèmes


### 🟢 Solutions