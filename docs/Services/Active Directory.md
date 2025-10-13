# Active Directory:

Creation OU pour chaque service disponible Production avec deux sousOU,Conception,Informatique

Ajout des utilistaeur dans chaque servies et dans chaque OU et dans chaque groupe 

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

