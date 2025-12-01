# Active Directory

---

## Organisation des unités (OU)
---

### Création d’une OU pour chaque service disponible :

- Production
- Sous-OU :

    - A
    - B

- Conception
- Informatique
---

## Gestion des utilisateurs
---
### Ajout des utilisateurs :

- Dans chaque service

- Dans chaque OU

- Dans chaque groupe associé
---
## Liste des Utilisateur:
---
| NOM | PRENOM | SERVICE | EQUIPE |
|-----|--------|---------|--------|
| CHEUVAN | Marie Alice | CONCEPTION | |
| PREHEM-TORTA | Pierric | CONCEPTION | |
| QUETEBOEUFS | Esmeralda | CONCEPTION | |
| BRAEUX | Salvatore | CONCEPTION | |
| REPSODE | Yann | CONCEPTION | |
| GHRASSA | Christiane | CONCEPTION | |
| NWONWU | Fatiha | CONCEPTION | |
| BROUERD | Fatiha | CONCEPTION | |
| BEYREMOGLU | Corinne | CONCEPTION | |
| MAOT | Denise | CONCEPTION | |
| LAMONE | John | CONCEPTION | |
| MOROGLU | Farid | CONCEPTION | |
| PEALLET | Nadège | CONCEPTION | |
| DEMECON | tante | CONCEPTION | |
| GOMES EDUERDO | Yvan | PRODUCTION | A |
| BENOAT | Vincent | PRODUCTION | A |
| NEAL | Erminda | PRODUCTION | A |
| SEBEE | Vincent | PRODUCTION | A |
| DEUPHAN | Laurence | PRODUCTION | A |
| HEMEDE | Yamina | PRODUCTION | A |
| DERON | Jérôme | PRODUCTION | A |
| MERGOT | Myriam | PRODUCTION | A |
| ROUSSEEU | Ali-Osman | PRODUCTION | A |
| CHEMBRAN | Sandrine | PRODUCTION | A |
| DJEDJE YOHOU | Joao Fernando | PRODUCTION | A |
| DAEBY | Virgil | PRODUCTION | A |
| BERBASSOU | Farid | PRODUCTION | A |
| QUETEBOEUFS | Roland | PRODUCTION | A |
| BECF | Khemais | PRODUCTION | A |
| CHRASTEAN | Yamina | PRODUCTION | A |
| DAMBEMBU | Marie Laur | PRODUCTION | A |
| FOFENE | Esther | PRODUCTION | A |
| BECF | Michelle | PRODUCTION | A |
| BOURET | Dalila | PRODUCTION | A |
| DZEBETOU-ECFO | Luis | PRODUCTION | A |
| AGHALEHRAZ | Cedric | PRODUCTION | A |
| PREHEM-TORTA | Viviane | PRODUCTION | A |
| SOLER | Jeanlou | PRODUCTION | A |
| DOUHEBA | Joao | PRODUCTION | A |
| PERASY | Frédérique | PRODUCTION | B |
| GRAGNON | Latha | PRODUCTION | B |
| BREENT MACHEL | Agnes | PRODUCTION | B |
| CHEPASEEU | Pierre Marc | PRODUCTION | B |
| ELAPOUR-EZEDEH | Benamar | PRODUCTION | B |
| RESOEVETSERE | Hélène | PRODUCTION | B |
| ELEXENDRE | Jeanine | PRODUCTION | B |
| DZEFEROVAC EUGUAN | Thérésia | PRODUCTION | B |
| BRAERE | Valerie | PRODUCTION | B |
| DOMANGUES VAEARE | Georges | PRODUCTION | B |
| GONZELVES | Abdelkader | PRODUCTION | B |
| LAVOLSA | Maïmouna | PRODUCTION | B |
| MEZERD | Jean-Eddy | PRODUCTION | B |
| GODOU | Lassaad | PRODUCTION | B |
| BLEASE | Kathia | PRODUCTION | B |
| BAGUEREEU | Marie Ange | PRODUCTION | B |
| LEDEUPHAN | Karine | PRODUCTION | B |
| EME | Marie-Luc | INFORMATIQUE | |
| DORENGE | Marine | INFORMATIQUE | |
| THARENT | Alvaro | INFORMATIQUE | |
| BENTELEB | John | INFORMATIQUE | |

---
## Script powershell utilisé:
---
```h
# Script d'importation d'utilisateurs Active Directory depuis CSV

 
# Importer le module Active Directory

Import-Module ActiveDirectory
 
# Définir les chemins de base de votre infrastructure

$domainDN = "DC=orl,DC=orleans,DC=sportludique,DC=fr"

$ouBasePath = "$domainDN"
 
# Définir les OU des services (sans sous-OU)

$ouServices = @{

    "Conception" = "OU=Conception,$ouBasePath"

    "Informatique" = "OU=Informatique,$ouBasePath"

    "Production" = "OU=Production,$ouBasePath"

}
 
# Définir les groupes principaux par service

$groupesServices = @{

    "Conception" = "GRP_CONCEPTION"

    "Informatique" = "GRP_INFORMATIQUE"

    "Production" = "GRP_PRODUCTION"

}
 
# Définir les sous-groupes Production

$sousGroupesProduction = @{

    "A" = "GRP_PRODUCTION_A"

    "B" = "GRP_PRODUCTION_B"

}
 
# Chemin du fichier CSV

$csvPath = ".\orleans.csv"
 
# Vérifier si le fichier CSV existe

if (-not (Test-Path $csvPath)) {

    Write-Host "Erreur : Le fichier $csvPath n'existe pas !" -ForegroundColor Red

    exit

}
 
# Importer les données du CSV

# Format attendu : NOM;PRENOM;SERVICE;SOUS_OU

$utilisateurs = Import-Csv -Path $csvPath -Delimiter ";" -Encoding UTF8
 
Write-Host "============================================" -ForegroundColor Cyan

Write-Host "Import d'utilisateurs dans Active Directory" -ForegroundColor Cyan

Write-Host "============================================" -ForegroundColor Cyan

Write-Host ""
 
foreach ($user in $utilisateurs) {

    $nom = $user.NOM.Trim()

    $prenom = $user.PRENOM.Trim()

    $service = $user.SERVICE.Trim()

    $sousOU = if ($user.SOUS_OU) { $user.SOUS_OU.Trim() } else { "" }

    # Vérifier que le service existe dans notre mapping

    if (-not $ouServices.ContainsKey($service)) {

        Write-Host "⚠ Utilisateur $prenom $nom : Service '$service' non reconnu. Ignoré." -ForegroundColor Yellow

        continue

    }

    # Construire les informations de l'utilisateur

    $samAccountName = ($prenom.Substring(0,1) + $nom).ToLower()

    $userPrincipalName = "$samAccountName@orl.orleans.sportludique.fr"

    $displayName = "$prenom $nom"

    # Déterminer le chemin OU (avec ou sans sous-OU)

    if ($sousOU -ne "") {

        $ouPath = "OU=$sousOU," + $ouServices[$service]

    }

    else {

        $ouPath = $ouServices[$service]

    }

    # Mot de passe par défaut (à changer lors de la première connexion)

    $defaultPassword = ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force

    try {

        # Vérifier si l'utilisateur existe déjà

        $existingUser = Get-ADUser -Filter "SamAccountName -eq '$samAccountName'" -ErrorAction SilentlyContinue

        if ($existingUser) {

            Write-Host "⚠ Utilisateur $samAccountName existe déjà. Ignoré." -ForegroundColor Yellow

        }

        else {

            # Créer l'utilisateur

            New-ADUser `

                -Name $displayName `

                -GivenName $prenom `

                -Surname $nom `

                -SamAccountName $samAccountName `

                -UserPrincipalName $userPrincipalName `

                -Path $ouPath `

                -AccountPassword $defaultPassword `

                -Enabled $true `

                -ChangePasswordAtLogon $true `

                -Department $service `

                -Description "Utilisateur du service $service"

            Write-Host "✓ Utilisateur créé : $displayName ($samAccountName)" -ForegroundColor Green

            # Ajouter au groupe principal du service

            $groupePrincipal = $groupesServices[$service]

            $groupePrincipalDN = "CN=$groupePrincipal," + $ouServices[$service]

            try {

                Add-ADGroupMember -Identity $groupePrincipalDN -Members $samAccountName -ErrorAction Stop

                Write-Host "  └─ Ajouté au groupe : $groupePrincipal" -ForegroundColor Cyan

            }

            catch {

                Write-Host "  └─ ⚠ Erreur ajout au groupe $groupePrincipal : $($_.Exception.Message)" -ForegroundColor Yellow

            }

            # Si service Production, ajouter aussi au sous-groupe

            if ($service -eq "Production" -and $sousOU -ne "") {

                if ($sousGroupesProduction.ContainsKey($sousOU)) {

                    $sousGroupe = $sousGroupesProduction[$sousOU]

                    $sousGroupeDN = "CN=$sousGroupe," + $ouServices[$service]

                    try {

                        Add-ADGroupMember -Identity $sousGroupeDN -Members $samAccountName -ErrorAction Stop

                        Write-Host "  └─ Ajouté au sous-groupe : $sousGroupe" -ForegroundColor Cyan

                    }

                    catch {

                        Write-Host "  └─ ⚠ Erreur ajout au sous-groupe $sousGroupe : $($_.Exception.Message)" -ForegroundColor Yellow

                    }

                }

            }

        }

    }

    catch {

        Write-Host "✗ Erreur lors de la création de $displayName : $($_.Exception.Message)" -ForegroundColor Red

    }

}
 
Write-Host ""

Write-Host "============================================" -ForegroundColor Cyan

Write-Host "Import terminé" -ForegroundColor Cyan

Write-Host "============================================" -ForegroundColor Cyan
```

# Active Directory Secondaire

## Étape 1 : Installation du Rôle AD DS

1. Ouvrez le **Gestionnaire de serveur**
2. Cliquez sur **Gérer** → **Ajouter des rôles et fonctionnalités**
3. Assistant d'ajout de rôles :
   - Page **Avant de commencer** : Cliquez sur **Suivant**
   - **Type d'installation** : Sélectionnez **Installation basée sur un rôle ou une fonctionnalité** → **Suivant**
   - **Sélection du serveur** : Choisissez votre serveur → **Suivant**
   - **Rôles de serveurs** : Cochez **Services AD DS (Active Directory Domain Services)**
   - Une fenêtre popup apparaît, cliquez sur **Ajouter des fonctionnalités** → **Suivant**
   - **Fonctionnalités** : Laissez par défaut → **Suivant**
   - **AD DS** : Lisez les informations → **Suivant**
   - **Confirmation** : Cliquez sur **Installer**
4. Attendez la fin de l'installation (ne fermez pas la fenêtre)

## Étape 2 : Promotion en Contrôleur de Domaine

1. À la fin de l'installation, cliquez sur **Promouvoir ce serveur en contrôleur de domaine** (lien avec drapeau jaune dans le Gestionnaire de serveur)

### Configuration du déploiement
- Sélectionnez **Ajouter un contrôleur de domaine à un domaine existant**
- Domaine : Vérifiez que votre domaine est affiché (ex : entreprise.local)
- Cliquez sur **Modifier** pour fournir les identifiants d'administrateur du domaine si nécessaire
- Cliquez sur **Suivant**

### Options du contrôleur de domaine
- Niveau fonctionnel : Laissez la valeur par défaut
- Cochez **Serveur DNS** (recommandé)
- Cochez **Catalogue global (GC)** (recommandé pour la redondance)
- Entrez un **mot de passe DSRM** (mode restauration des services d'annuaire) - À conserver précieusement !
- Cliquez sur **Suivant**

### Options DNS
- Ignorez l'avertissement éventuel sur la délégation DNS
- Cliquez sur **Suivant**

### Options supplémentaires
- Le système détecte automatiquement le DC source pour la réplication
- Vérifiez que le bon DC est sélectionné
- Cliquez sur **Suivant**

### Vérification de la configuration requise
- L'assistant effectue une vérification complète
- Lisez les avertissements éventuels (certains sont normaux)
- Si tout est OK, cliquez sur **Installer**

Le serveur redémarrera automatiquement après l'installation.

## Étape 3 : Vérification Post-Installation

Après le redémarrage, connectez-vous avec un compte administrateur du domaine.

### Vérification dans le Gestionnaire de serveur
1. Ouvrez le **Gestionnaire de serveur**
2. Vérifiez que le rôle **AD DS** est affiché
3. Aucune erreur ne devrait apparaître

### Vérification de la réplication
1. Ouvrez **Outils** → **Sites et services Active Directory**
2. Naviguez vers : **Sites** → **Default-First-Site-Name** → **Servers** → Votre nouveau DC
3. Vérifiez la présence des connexions de réplication

### Test avec PowerShell
Ouvrez PowerShell en tant qu'administrateur et exécutez :

```powershell

# Vérifier la réplication
repadmin /replsummary

```

## Configuration DNS Recommandée

1. Sur le DC principal, ouvrez **Gestionnaire DNS**
2. Ajoutez l'adresse IP du nouveau DC comme serveur DNS secondaire
3. Sur les clients, configurez le nouveau DC comme DNS secondaire

## Sauvegarde du Mot de Passe DSRM

⚠️ **IMPORTANT** : Conservez le mot de passe DSRM dans un coffre-fort sécurisé. Il est nécessaire pour la restauration en cas de problème grave.


# Documentation - Intégration Active Directory avec Proxmox VE


## 1. Vue d'ensemble

### Objectif
Permettre aux utilisateurs Active Directory de s'authentifier sur Proxmox VE avec leurs identifiants AD et gérer les permissions via des groupes AD.

---

## 2. Prérequis

### 2.1 Informations Active Directory nécessaires

Collectez les informations suivantes avant de commencer :
```
Domaine AD         : orl.orleans.sportludique.fr
Serveur DC         : dc-01.orleans.sportludique.fr
Adresse IP DC      : 172.28.120.1
Base DN            : DC=sportludique,DC=fr
Compte de liaison  : cn=proxmox-bind,cn=Users,dc=orl.orleans.sportludique,dc=fr
Mot de passe       : *******
Port LDAP          : 389 
```

### 2.2 Groupes AD à créer

Créez ces groupes dans AD pour organiser les accès :

| Groupe AD | Rôle Proxmox | Permissions |
|-----------|--------------|-------------|
| PVE-Administrators | Administrator | Accès complet |


---

## 3. Configuration Active Directory

### 3.1 Créer le compte de service

**Via PowerShell (recommandé) :**
```powershell
# Créer le compte de service
New-ADUser -Name "proxmox-" `
    -UserPrincipalName "proxmox.l" `
    -SamAccountName "proxmox-bind" `
    -Path "CN=Users,DC=exemple,DC=local" `
    -AccountPassword (ConvertTo-SecureString "Azerty1234!" -AsPlainText -Force) `
    -Description "Compte de liaison LDAP pour Proxmox VE"

# Vérifier la création
Get-ADUser -Identity proxmox-bind
```

**Via Interface graphique :**

1. Ouvrir **Active Directory Users and Computers**
2. Clic droit sur **Users** → **New** → **User**
3. Remplir les informations :
   - First name : `proxmox`
   - User logon name : `proxmox`
4. Définir un mot de passe fort
5. Cocher :
   - ✅ Password never expires
   - ✅ User cannot change password
6. Cliquer **Finish**

### 4.2 Créer les groupes de sécurité

**Via PowerShell :**
```powershell
# Créer une OU dédiée (optionnel mais recommandé)
New-ADOrganizationalUnit -Name "Proxmox" -Path "DC=orl.orleans.sportludique,DC=fr"

# Créer les groupes
$groups = @(
    @{Name="PVE-Administrators"; Description="Administrateurs Proxmox - Accès complet"},
    @{Name="PVE-PowerUsers"; Description="Power Users Proxmox - Gestion VMs"},
    @{Name="PVE-Users"; Description="Utilisateurs Proxmox - Utilisation VMs"},
    @{Name="PVE-Auditors"; Description="Auditeurs Proxmox - Lecture seule"}
)

foreach ($group in $groups) {
    New-ADGroup -Name $group.Name `
        -GroupScope Global `
        -GroupCategory Security `
        -Path "OU=Proxmox,DC=orl.orleans.sportludique,DC=fr" `
        -Description $group.Description
}

# Vérifier la création
Get-ADGroup -Filter 'Name -like "PVE-*"' | Select-Object Name, DistinguishedName
```

### 4.3 Ajouter des utilisateurs aux groupes

**Via PowerShell :**
```powershell
# Ajouter un utilisateur à un groupe
Add-ADGroupMember -Identity "PVE-Administrators" -Members "jdupont"

# Ajouter plusieurs utilisateurs
Add-ADGroupMember -Identity "PVE-Users" -Members "jdupont","mmartin","sdurand"

# Vérifier les membres
Get-ADGroupMember -Identity "PVE-Administrators" | Select-Object Name
```

---

## 5. Configuration Proxmox VE

### 5.2 Configuration via interface web

#### Méthode 1 : Interface graphique (recommandé)

1. **Se connecter à Proxmox**
   - URL : `https://[IP-PROXMOX]:8006`
   - Login : `root@pam`

2. **Accéder à la configuration**
   - Datacenter → Permissions → Realms
   - Cliquer sur **Add**

3. **Choisir le type**
   - Sélectionner **LDAP** (ou **Active Directory** pour méthode native)

4. **Remplir les paramètres LDAP**
```
Realm              : ad
Type               : LDAP
Server             : dc01.orl.orleans.sportludique.fr
Port               : 389 (LDAP) 
SSL                : (LDAP)
Base DN            : DC=orl.orleans.sportludique,DC=fr
User Attribute     : sAMAccountName
Bind User          : cn=proxmox,cn=Users,dc=orl.orleans.sportludique,dc=fr
Bind Password      : Azerty1234!
```

5. **Tester la connexion**
   - Cliquer sur **Test**
   - Vérifier que "Connection successful" s'affiche

6. **Enregistrer**
   - Cliquer sur **Add**

#### Méthode 2 : Ligne de commande

**Configuration LDAP simple :**
```bash
# Se connecter en SSH à Proxmox
ssh root@192.168.45.20

# Ajouter le realm LDAP
pveum realm add ad --type ldap \
  --server dc01.orl.orleans.sportludique.fr \
  --port 389 \
  --base-dn "DC=orl.orleans.sportludique,DC=fr" \
  --user-attr sAMAccountName \
  --bind-dn "cn=proxmox-bind,cn=Users,dc=orl.orleans.sportludique,dc=fr" \
  --password "Azerty1234!" \
  --filter "(objectClass=person)"

# Vérifier la configuration
pveum realm list
cat /etc/pve/domains.cfg
```

### 5.3 Synchroniser les groupes AD

**Via interface web :**

1. **Accéder aux groupes**
   - Datacenter → Permissions → Groups
   - Cliquer sur **Create**

2. **Créer un groupe Proxmox lié à AD**
```
Group ID    : ad-administrators
Comment     : Administrateurs AD
```

3. **Répéter pour chaque groupe**

**Via ligne de commande :**
```bash
# Créer les groupes Proxmox
pveum group add ad-administrators -comment "Administrateurs AD"
pveum group add ad-powerusers -comment "Power Users AD"
pveum group add ad-users -comment "Utilisateurs AD"
pveum group add ad-auditors -comment "Auditeurs AD"

# Vérifier
pveum group list
```

### 5.4 Configuration avancée

**Activer la synchronisation automatique des groupes :**
```bash
# Éditer le fichier de configuration
nano /etc/pve/domains.cfg

# Ajouter ces paramètres au realm
ldap: ad
    base_dn DC=orl.orleans.sportludique,DC=fr
    bind_dn cn=proxmox-bind,cn=Users,dc=orl.orleans.sportludique,dc=fr
    port 389
    server dc01.orl.orleans.sportludique.fr
    user_attr sAMAccountName
    sync_attributes memberof
    sync-defaults-options scope=both,enable-new=1
    filter (objectClass=person)
```

**Configurer la synchronisation des groupes AD :**
```bash
# Créer un script de synchronisation
cat > /usr/local/bin/sync-ad-groups.sh << 'EOF'
#!/bin/bash
# Script de synchronisation des groupes AD avec Proxmox

REALM="ad"
AD_GROUPS=("PVE-Administrators" "PVE-PowerUsers" "PVE-Users" "PVE-Auditors")

for group in "${AD_GROUPS[@]}"; do
    echo "Synchronisation du groupe: $group"
    pveum sync $REALM --scope both --enable-new 1 --full --purge 0
done

echo "Synchronisation terminée"
EOF

# Rendre le script exécutable
chmod +x /usr/local/bin/sync-ad-groups.sh

# Tester le script
/usr/local/bin/sync-ad-groups.sh

# Ajouter au cron (toutes les heures)
crontab -e
# Ajouter cette ligne :
0 * * * * /usr/local/bin/sync-ad-groups.sh >> /var/log/ad-sync.log 2>&1
```

---

## 6. Gestion des permissions

### 6.1 Attribution des rôles

**Rôles Proxmox disponibles :**

| Rôle | Permissions | Utilisation recommandée |
|------|-------------|-------------------------|
| **Administrator** | Toutes permissions | Administrateurs système |
| **PVEAdmin** | Gestion VMs, stockage, réseau | Power users / équipes IT |
| **PVEVMAdmin** | Gestion VMs complète | Administrateurs VMs |
| **PVEVMUser** | Démarrer/arrêter VMs | Utilisateurs finaux |
| **PVEAuditor** | Lecture seule | Auditeurs / monitoring |
| **PVEDatastoreUser** | Accès stockage | Utilisateurs backup |

---

## 7. Tests et validation

### 7.1 Test d'authentification

**Test 1 : Connexion via interface web**

1. Se déconnecter de Proxmox (logout)
2. Aller sur `https://[IP-PROXMOX]:8006`
3. Sélectionner le realm **ad** dans le menu déroulant
4. Entrer les identifiants AD :
   - Username : `[identifiant AD]` 
   - Password : `[mot de passe AD]`
   - Realm : **ad**
5. Cliquer sur **Login**


---
