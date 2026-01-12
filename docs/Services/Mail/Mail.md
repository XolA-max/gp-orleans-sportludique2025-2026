# HMAILSERVER – Installation et Configuration

---

## 1. Prérequis

### Environnement requis
*   Machine virtuelle ou physique sous **Windows 10 ou Windows 11**
*   Accès administrateur à la machine
*   Connexion réseau fonctionnelle

### Téléchargement
Téléchargez hMailServer et .NET Framework 2.0 SP2 (requis).

!!! info "Téléchargements"
    *   [hMailServer](https://www.hmailserver.com/download)
    *   [.NET Framework 2.0 SP2 (x64)](https://www.microsoft.com/en-us/download/details.aspx?id=9834)

---

---

## 2. Installation de hMailServer

### 2.1 Exécution se l'installateur

1.  Lancez le fichier `.exe`.
2.  Selectionnez **Full installation**.
3.  Choisissez le moteur de base de données :
    *   **Use built-in database (simple)** (recommandé pour test).
    *   **External database** (pour production).
4.  Définissez le **mot de passe administrateur hMailServer**.

### 2.2 Accès à l'interface

*   Ouvrez **hMailServer Administrator**.
*   Connectez-vous avec le mot de passe défini.

---

---

## 3. Configuration des Domaines

### 3.1 Ajouter un domaine

1.  **Domains** → **Add**.
2.  Entrez le nom de domaine (ex: `orleans.sportludique.fr`).

### 3.2 Définir le domaine par défaut

1.  **Settings** → **Advanced** → **Default domain**.
2.  Sélectionnez votre domaine.

---

---

## 4. Création des Comptes E-mail

1.  **Domains** → *votre_domaine* → **Accounts**.
2.  Cliquez sur **Add**.
3.  Remplir :
    *   **Address** : `contact` (pour `contact@orleans.sportludique.fr`)
    *   **Password** : Définir un mot de passe.

---

---

## 5. Sécurité et Pare-feu Windows

### 5.1 Règles de Trafic Entrant (Inbound)

!!! info "Ouvrir le Pare-feu"
    Exécuter `wf.msc`

Créez une nouvelle règle pour les ports TCP suivants :

| Port | Service | Description |
| ---- | ------- | ----------- |
| 25 | SMTP | Envoi de mail (Non sécurisé) |
| 587 | SMTP | Envoi de mail (STARTTLS) |
| 143 | IMAP | Réception (Non sécurisé) |
| 993 | IMAPS | Réception (Sécurisé) |

### 5.2 Règles de Trafic Sortant (Outbound)

Autorisez uniquement la sortie pour **SMTP (25, 587)**.

---

---

## 6. Configuration Réseau

### Route Statique

Pour permettre la communication avec le réseau LAN si nécessaire.

!!! info
    ```cmd
    route add -p [reseau] MASK [masque] [passerelle]
    ```

    Vérification :
    ```cmd
    route print
    ```

---

---

## 7. Configuration DNS (Bind9)

### 7.1 Zone Externe

Fichier : `/etc/bind/db.orleans.sp.fr.externe`

```dns
@   IN  MX  10 smtp.orleans.sportludique.fr.
smtp IN A   183.44.45.1  ; IP Publique du routeur
```

### 7.2 Zone Interne

Fichier : `/etc/bind/db.orleans.sp.fr.interne`

```dns
@   IN  MX  10 smtp.orleans.sportludique.fr.
mail IN A   192.168.45.7 ; IP du serveur hMail
smtp IN CNAME mail
imap IN CNAME mail
```

---

---

## 8. Liaison avec Active Directory

Un script PowerShell permet de synchroniser les utilisateurs AD vers hMailServer.

[Voir le script de synchronisation](scriptHmail.md)

---

---

## 9. Configuration Client (Thunderbird)

### Paramètres

| Type | Hôte | Port | Sécurité |
| ---- | ---- | ---- | -------- |
| **IMAP** | `imap.orleans.sportludique.fr` | 143 (ou 993) | STARTTLS (si 993) |
| **SMTP** | `smtp.orleans.sportludique.fr` | 587 (ou 25) | STARTTLS |

---

---

## 10. Routeur / NAT

N'oubliez pas les règles de redirection de port (PAT) sur le routeur :

*   Port 25 (WAN) → Port 25 (LAN Serveur Mail)
*   Port 587 (WAN) → Port 587 (LAN Serveur Mail)
