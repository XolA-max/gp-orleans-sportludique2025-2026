# HMAILSERVER – Documentation d’installation et de configuration

---

## 1. Prérequis

### 1.1 Environnement requis
- Machine virtuelle ou physique sous **Windows 10 ou Windows 11**
- Accès administrateur à la machine
- Connexion réseau fonctionnelle (LAN / Internet selon le contexte)

### 1.2 Téléchargement de hMailServer
Téléchargez la dernière version stable depuis le site officiel :  
👉 https://www.hmailserver.com/download

### 1.3 Installation de Microsoft .NET Framework 2.0 SP2 (x64)
> hMailServer nécessite ce composant. Sans lui, un message d’erreur apparaîtra lors de l’installation.

Lien de téléchargement :  
👉 https://www.microsoft.com/en-us/download/details.aspx?id=9834

---

## 2. Installation de hMailServer

### 2.1 Exécution de l’installateur
Lancez le fichier **.exe** téléchargé.

1. Acceptez la licence
2. Sélectionnez **Full installation**
3. Choisissez le moteur de base de données :
   - **Use built-in database (simple)** → recommandé pour les tests et petits environnements
   - **External database** → MySQL / MariaDB (environnements de production)
4. Définissez un **mot de passe administrateur hMailServer**
5. Terminez l’installation

### 2.2 Accès à l’interface d’administration
- Ouvrez **hMailServer Administrator**
- Connectez-vous avec le mot de passe défini précédemment

---

## 3. Configuration des domaines

### 3.1 Ajouter un domaine
1. Dans le panneau de gauche, cliquez sur **Domains**
2. Cliquez sur **Add**
3. Entrez votre nom de domaine, par exemple :
   - `orleans.sportludique.fr`
   - `mon_domaine.com`

### 3.2 Définir le domaine par défaut
1. Allez dans **Settings**
2. **Advanced → Default domain**
3. Sélectionnez votre domaine (ex. `orleans.sportludique.fr`)

---

## 4. Création des comptes e-mail

1. Accédez à :  
   **Domains → orleans.sportludique.fr → Accounts**
2. Cliquez sur **Add…**
3. Renseignez :
   - Nom du compte : `contact`
   - Mot de passe

📧 Adresse créée : `contact@orleans.sportludique.fr`

---

## 5. Sécurité et pare-feu Windows

### 5.1 Règles de trafic entrant (Inbound Rules)
Accéder au pare-feu Windows :
```cmd
wf.msc
```

Ports à autoriser (TCP) :
- **143** → IMAP (non sécurisé)
- **993** → IMAPS (sécurisé)
- **25** → SMTP (non sécurisé)
- **587** → SMTP avec STARTTLS

Procédure :
1. Inbound Rules → New Rule…
2. Type : **Port**
3. Protocole : **TCP**
4. Ports spécifiques : `25,143,587,993`
5. Action : **Allow the connection**
6. Profils : Domain / Private / Public (selon le contexte)
7. Nom : *Autoriser IMAP / SMTP (secure & non-secure)*

### 5.2 Règles de trafic sortant (Outbound Rules)
Même procédure que pour l’entrant, mais uniquement pour :
- **25**
- **587**

> Seul le protocole SMTP doit sortir du serveur

---

## 6. Configuration réseau – Route statique

Permet la communication avec le réseau LAN.

Commande :
```cmd
route add -p [reseau] MASK [masque] [passerelle]
```

Afficher les routes existantes :
```cmd
route print
```

---

## 7. Configuration DNS

### 7.1 Fonctionnement des enregistrements MX
- Le **MTA (Mail Transfer Agent)** interroge les enregistrements MX
- Le DNS retourne les serveurs de messagerie avec leur priorité
- Le MTA tente la livraison SMTP du plus prioritaire au moins prioritaire

---

### 7.2 Zone DNS externe (BIND)
**Fichier :** `/etc/bind/db.orleans.sp.fr.externe`
```dns
$TTL    604800
@   IN  SOA ns1.orleans.sportludique.fr. root.orleans.sportludique.fr. (
        2025150938 604800 86400 2419200 604800 )
@   IN  NS  ns1.orleans.sportludique.fr.
@   IN  NS  ns2.orleans.sportludique.fr.
@   IN  A   183.44.45.1
ns1 IN  A   183.44.45.1
ns2 IN  A   183.44.45.1
www IN  A   183.44.45.1
smtp IN A   183.44.45.1
@   IN  MX  10 smtp.orleans.sportludique.fr.
```

✔ `smtp` pointe vers l’IP publique du routeur
✔ MX priorité 10 vers `smtp`

---

### 7.3 Zone DNS interne (BIND)
**Fichier :** `/etc/bind/db.orleans.sp.fr.interne`
```dns
$TTL    604800
@   IN  SOA ns1.orleans.sportludique.fr. root.orleans.sportludique.fr. (
        2025111315 604800 86400 2419200 604800 )
@   IN  NS  ns1.orleans.sportludique.fr.
ns1 IN  A   192.168.45.2
www IN  A   192.168.45.3
mail IN A   192.168.45.7
smtp IN CNAME mail
imap IN CNAME mail
@   IN  MX  10 smtp.orleans.sportludique.fr.
```

✔ `mail` pointe vers le serveur hMailServer
✔ `smtp` et `imap` sont des alias

---

## 8. Liaison avec Active Directory

Un script peut être utilisé pour importer automatiquement les utilisateurs AD.

📎 cliquer ici : [accedder à la page de script](scriptHmail.md)

---

## 9. Configuration d’un client Thunderbird

### 9.1 Informations utilisateur
- Nom : Jean
- Adresse : jean@orleans.sportludique.fr
- Mot de passe : ********

### 9.2 Serveur entrant (IMAP)
- Hôte : `imap.orleans.sportludique.fr`
- Port : 143 (non sécurisé) ou 993 (sécurisé)
- Sécurité : STARTTLS (si 993)
- Authentification : Mot de passe normal

### 9.3 Serveur sortant (SMTP)
- Hôte : `smtp.orleans.sportludique.fr`
- Port : 25 ou 587
- Sécurité : STARTTLS (recommandé)
- Authentification : Mot de passe normal

---

## 10. Routeur / NAT

Redirections nécessaires :
- **25 → SMTP**
- **587 → SMTP sécurisé (STARTTLS)**

---

## 11. Problèmes courants

### 11.1 Erreurs après changement de configuration

Solution (Thunderbird) :
1. Paramètres
2. Paramètres de compte
3. Dossiers locaux
4. Supprimer les données locales
5. Recréer le compte

---

📘 *Documentation améliorée et prête à être utilisée comme fichier Markdown (.md)*
