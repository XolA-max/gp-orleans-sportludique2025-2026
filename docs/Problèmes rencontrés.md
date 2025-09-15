# Compte-rendu des Séances

---

## 🗓️ Semaine 1

### 🔴 Problèmes
- Enorme latence lors de l’écriture sur les terminaux.  
- Les lettres ne correspondaient pas → beaucoup d’erreurs de saisie.  

### 🟢 Solutions
- Utilisation d’un PC séparé avec un câble bien tendu (pas de solution précise trouvée).  
- Connexions **SSH à distance** depuis mon ordinateur.  

---

## 🗓️ Semaine 2

### 🔴 Problèmes
- Problème de **liaison Trunk** entre le switch et le PC.  
- **Mauvaise configuration DNS** :  
  - Création d’un nouveau DNS.  
  - Suppression du DNS initial (lié au compte Administrateur).  
  - Suppression de la VM par erreur.  

### 🟢 Solutions
- Arrêter d’essayer d’établir une liaison entre les deux appareils.  
- Ne pas supprimer le DNS principal et bien le configurer dès le départ.  
- ⚠️ **Cloner la VM** pour éviter de devoir refaire toute l’installation.  

---

## 🗓️ Semaine 3

### 🔴 Problèmes & 🟢 Solutions

| Problème | Solution |
|----------|----------|
| **Interconnexion entre le switch et le routeur (VLAN 249)** | Créer le VLAN `249` et **le mettre en mode Access** sur une interface pour l’activer. Sinon, il reste inactif et ne peut pas être relié à une liaison Trunk. <br> **Commande** :<br>```bash<br>conf t<br>vlan 249<br>description Interco<br>``` |
| **ACL : mauvaise configuration** | (À compléter — problème rencontré mais pas encore corrigé) |

---

✅ **Test effectué** : VLAN actif après configuration.

