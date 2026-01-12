# Active Directory

---

## Organisation des Unités (OU)

### Structure Hiérarchique

*   **Production**
    *   Sous-OU :
        *   A
        *   B
*   **Conception**
*   **Informatique**

---

---

## Gestion des Utilisateurs

### Politique d'ajout

*   Dans chaque service.
*   Dans chaque OU.
*   Dans chaque groupe associé.

---

## Liste des Utilisateurs

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

---

## Automatisation PowerShell

### Script d'Importation des Utilisateurs

!!! info "Script d'import CSV vers AD"
    ```powershell
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

---

---

# Active Directory Secondaire

## Étape 1 : Installation du Rôle AD DS

1.  Ouvrez le **Gestionnaire de serveur**.
2.  Cliquez sur **Gérer** → **Ajouter des rôles et fonctionnalités**.
3.  Suivez l'assistant d'ajout de rôles :
    *   **Type d'installation** : Installation basée sur un rôle ou une fonctionnalité.
    *   **Rôles de serveurs** : Cochez **Services AD DS (Active Directory Domain Services)**.
4.  Attendez la fin de l'installation.

## Étape 2 : Promotion en Contrôleur de Domaine

1.  Cliquez sur le drapeau jaune (Notifications) → **Promouvoir ce serveur en contrôleur de domaine**.

### Configuration du déploiement
*   Sélectionnez **Ajouter un contrôleur de domaine à un domaine existant**.
*   **Domaine** : `orleans.sportludique.fr` (Vérifiez le nom).
*   Cliquez sur **Modifier** pour fournir les identifiants Admin.

### Options du contrôleur de domaine
*   Cochez **Serveur DNS** et **Catalogue global (GC)**.
*   Définissez le **mot de passe DSRM**.

### Options supplémentaires
*   Réplication depuis : **Any domain controller** (ou sélectionnez le DC principal spécifique).

### Vérification Post-Installation

!!! info "Tester la réplication via PowerShell"
    ```powershell
    repadmin /replsummary
    ```

## Configuration DNS Recommandée

!!! info
    1.  Sur le DC principal, ajoutez l’IP du nouveau DC comme serveur DNS secondaire.
    2.  Sur les clients, configurez le nouveau DC comme DNS secondaire.

!!! warning "Sauvegarde DSRM"
    Conservez le mot de passe **DSRM** en lieu sûr.

---

---

# Intégration Active Directory avec Proxmox VE

## 1. Vue d'ensemble

Permettre aux utilisateurs Active Directory de s'authentifier sur Proxmox VE.

## 2. Prérequis

### Informations Active Directory

!!! info "Paramètres de Connexion"
    ```text
    Domaine AD         : orl.orleans.sportludique.fr
    Serveur DC         : dc-01.orleans.sportludique.fr
    Adresse IP DC      : 172.28.120.1
    Base DN            : DC=sportludique,DC=fr
    Compte de liaison  : cn=proxmox-bind,cn=Users,dc=orl.orleans.sportludique,dc=fr
    Port LDAP          : 389 
    ```

### Groupes AD à créer

| Groupe AD | Rôle Proxmox | Permissions |
|-----------|--------------|-------------|
| **PVE-Administrators** | Administrator | Accès complet |
| **PVE-PowerUsers** | PVEAdmin | Gestion VMs |
| **PVE-Users** | PVEVMUser | Utilisation VMs |

---

## 3. Configuration Active Directory

### Création des groupes et utilisateurs de service

!!! info "PowerShell"
    ```powershell
    # Créer le compte de service
    New-ADUser -Name "proxmox-bind" `
        -UserPrincipalName "proxmox.bind@orl.orleans.sportludique.fr" `
        -SamAccountName "proxmox-bind" `
        -Path "CN=Users,DC=orl,DC=orleans,DC=sportludique,DC=fr" `
        -AccountPassword (ConvertTo-SecureString "Azerty1234!" -AsPlainText -Force) `
        -Description "Compte de liaison LDAP pour Proxmox VE"

    # Créer les groupes
    New-ADGroup -Name "PVE-Administrators" -GroupScope Global -GroupCategory Security
    New-ADGroup -Name "PVE-Users" -GroupScope Global -GroupCategory Security
    ```

---

## 5. Configuration Proxmox VE

### Configuration via interface web

1.  **Datacenter → Permissions → Realms** → **Add** → **LDAP**
2.  Remplir les champs :
    *   **Realm** : `ad`
    *   **Base DN** : `DC=orl,DC=orleans,DC=sportludique,DC=fr`
    *   **User Attribute** : `sAMAccountName`
    *   **Bind User** : `cn=proxmox-bind,cn=Users,dc=orl...`

### Configuration via Ligne de commande

!!! info
    `ssh root@proxmox`

    ```bash
    pveum realm add ad --type ldap \
      --server dc01.orl.orleans.sportludique.fr \
      --port 389 \
      --base-dn "DC=orl,DC=orleans,DC=sportludique,DC=fr" \
      --user-attr sAMAccountName \
      --bind-dn "cn=proxmox-bind,cn=Users,dc=orl..." \
      --password "Azerty1234!" \
      --filter "(objectClass=person)"
    ```

### Synchronisation des Groupes

!!! info
    Création d'un script de synchronisation automatique :
    
    `nano /usr/local/bin/sync-ad-groups.sh`

    ```bash
    #!/bin/bash
    REALM="ad"
    AD_GROUPS=("PVE-Administrators" "PVE-Users")

    for group in "${AD_GROUPS[@]}"; do
        echo "Syncing group: $group"
        pveum group add $group --comment "Synced from AD" || true
        pveum sync $REALM --scope both --enable-new 1 --full --purge 0
    done
    ```

---

## 6. Gestion des permissions

| Rôle | Permissions | Utilisation recommandée |
|------|-------------|-------------------------|
| **Administrator** | Toutes permissions | Administrateurs système |
| **PVEAdmin** | Gestion VMs, stockage, réseau | Power users / équipes IT |
| **PVEVMUser** | Démarrer/arrêter VMs | Utilisateurs finaux |
| **PVEAuditor** | Lecture seule | Auditeurs / monitoring |

---

