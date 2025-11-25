# HMAIL
## 1. Prérequis

---
#### VM Windows 10 ou 11 Prête à l'emplois
### Installation de HMAIL
```
Aller sur le site officiel :
https://www.hmailserver.com/download
```
---

###  Installation de Microsoft .NET Framework 2.0 Service Pack 2 (x64)
> Car un message d'erreur va arriver nous disant qu'il faut installer ce fichier  
```
https://www.microsoft.com/en-us/download/details.aspx?id=9834
```

---



---


### 2. Exécutez le fichier .exe

```
Acceptez la licence
Choisissez : Full installation 
```
```
Moteur de base de données :

Use built-in database (simple)
ou External database (si vous utilisez MySQL / MariaDB)

```
```
Définissez un mot de passe d’administration pour hMailServer.

Terminez l’installation.
```

>Ouvrez hMailServer Administrator et connecter vous

---
### 2.1 Configuration du domaine

- Ajouter un domaine

- Dans le panneau de gauche, cliquez sur Domains

- Cliquez sur Add

- Entrez votre domaine (exemples) :
- ```orleans.sportludique.fr``` ou ```[mon_domaine_.com]```



#### Puis mettre un domaine par défaut :
- Settings 


- Advanced -> Default domain : ```orleans.sportludique.fr``` ou ```[mon_domaine_.com]```


### 2.2 Création d’un compte e-mail

---


- Dans Domains → orleans.sportludique.fr → 

- Accounts --  Add…

Entrez le nom de boîte (ex : contact)

Cela donne : contact@orleans.sportludique.fr

Définissez un mot de passe



---

## 3. sécurité
## 3.1 Créer les règles de trafic entrant (inbound)
>Sur le Pare-feu windows 

    pour y acceder dans le cmd:  wf.msc
                                        


> Ajouter des règles pour autoriser les ports des protocoles IMAP SMTP et leur version sécurisé 
- Port TCP 143 (Non sécurisé - IMAP) 
- Port TCP 993 (Sécurisé - IMAPS) 
- Port TCP 25 (Non sécurisé- SMTP) 
- Port TCP 587 (STARTTLS) 

Dans la colonne de gauche, cliquer sur Inbound Rules

    - À droite → New Rule…
    - Choisir Port  
    - Choisir TCP
    - Select Specific local ports
    - Entrer les ports voulus : 25,143,587,993
    - Choisir Allow the connection
    - Cocher les trois profils : Domain, Private, Public (ou adapter selon votre réseau)
    - Donner un nom : autoriser écoute sur les port IMAP et SMTP et version secure

## 3.2 Créer les règles de trafic sortant (Outbound)

>Faire la même manipulation mais mettre seulement les ports 25 et 587 car seul le protocole SMTP doit sortir 

## 3.3 Créer une route statique (Windows)
> Pour aller dans le réseau LAN et pouvoir communiquer

```
route add -p [réseau_à_joindre] MASK [masque] [passerelle]
```
> -p pour la rendre permanent

#### pour voir les routes 
    route print
---

 ## 4. Configuration DNS

 #### Quel est le processus d'interrogation d'un enregistrement MX ?

- Le logiciel *d'agent de transfert de messages (MTA)* est responsable de l'interrogation des enregistrements MX. 
- Lorsqu'un utilisateur *envoie un e-mail*, le MTA envoie une requête DNS pour identifier les serveurs de messagerie des destinataires de l'e-mail.
- Le MTA établit une connexion SMTP avec ces serveurs de messagerie, en commençant par les domaines prioritaires 

 #### 4.1 configurtion /etc/bind/db.orleans.sp.fr.externe      
 ```
 ;
 ; BIND data file for local loopback interface
 ;
 $TTL    604800
 @       IN      SOA   ns1.orleans.sportludique.fr. root.orleans.sportludique.fr. (
                     2025150938         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
 ;
 @       IN      NS      ns1.orleans.sportludique.fr.
 @       IN      NS      ns2.orleans.sportludique.fr.

 @       IN      A       183.44.45.1
 ns1     IN      A       183.44.45.1
 ns2     IN      A       183.44.45.1
 www     IN      A       183.44.45.1
 wp      IN      A       183.44.45.1
 smtp    IN      A       183.44.45.1
        IN      MX      10      smtp.orleans.sportludique.fr
```
- on créer un enregistrement nommé smtp qui pointe vers le routeur extérieur
- on créer un enregistrement MX priorité 10 qui pointe 

#### 4.2 configurtion /etc/bind/db.orleans.sp.fr.interne
 ```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     ns1.orleans.sportludique.fr. root.orleans.sportludique.fr. (
                         2025111315   ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns1.orleans.sportludique.fr.
ns1     IN      A       192.168.45.2
www     IN      A       192.168.45.3
mail    IN      A       192.168.45.7
        IN      MX      10      smtp.orleans.sportludique.fr    
smtp    IN      CNAME   mail
imap    IN      CNAME   mail

 ```
- on créer un enregistrement nommé mail qui pointe vers le serveur mail
- puis deux enregistrement smtp et imap qui pointent vers mail avec le CNAME
- on créer un enregistrement MX priorité 10 qui pointe vers smtp.orleans.sportludique.fr 


---

## 5. Lier les Utilisateurs de L'AD  

>Exemple de script qui peux être utiliser pour ajouter les utilisateurs 
```
# Script de synchronisation Active Directory vers hMailServer
# Support AD distant avec authentification

# ========== CONFIGURATION ==========
# Configuration hMailServer
$hMailAdminPassword = "Admin" #Mot de Passe de l'admin
$defaultDomain = "orleans.sportludique.fr" 
$defaultPassword = "etudiant" # Mot de passe par défaut pour nouveaux comptes

# Configuration Active Directory DISTANT
$adServer = "172.28.120.1"  # Nom ou IP du serveur AD
$adDomain = "orl.orleans.sportludique.fr"# Domaine AD
$adUsername = "Administrateur@orl.orleans.sportludique.fr"  # Utilisateur avec droits de lecture AD
$adPassword = "P@ssw0rd123456!"  # Mot de passe pour l'AD

# Construction du chemin LDAP
$ldapPath = "LDAP://$adServer/DC=orl,DC=orleans,DC=sportludique,DC=fr"

# Options
$createNewAccounts = $true
$updateExistingAccounts = $true
$accountMaxSize = 1024  # Taille max en MB
$enableAccountsByDefault = $true

Write-Host "========================================" -ForegroundColor Cyan
Write-Host "Synchronisation AD -> hMailServer" -ForegroundColor Cyan
Write-Host "========================================" -ForegroundColor Cyan

# ========== TEST CONNECTIVITÉ AD ==========
Write-Host "`n[Préparation] Test de connectivité AD..." -ForegroundColor Yellow

try {
    Write-Host "  → Test ping vers $adServer..." -ForegroundColor Gray
    $pingResult = Test-Connection -ComputerName $adServer -Count 1 -Quiet -ErrorAction SilentlyContinue
    
    if ($pingResult) {
        Write-Host "  ✓ Serveur AD accessible" -ForegroundColor Green
    } else {
        Write-Host "  ⚠ Serveur AD non pingable (peut être normal si ICMP bloqué)" -ForegroundColor Yellow
    }
    
    Write-Host "  → Test port LDAP 389..." -ForegroundColor Gray
    $tcpClient = New-Object System.Net.Sockets.TcpClient
    $asyncResult = $tcpClient.BeginConnect($adServer, 389, $null, $null) #changer le port pour mettre LDAPs:636 
    $wait = $asyncResult.AsyncWaitHandle.WaitOne(3000, $false)
    
    if ($wait) {
        $tcpClient.EndConnect($asyncResult)
        $tcpClient.Close()
        Write-Host "  ✓ Port LDAP 389 accessible" -ForegroundColor Green
    } else {
        Write-Host "  ✗ Port LDAP 389 non accessible" -ForegroundColor Red
        Write-Host "    Vérifiez le pare-feu et la connectivité réseau" -ForegroundColor Yellow
    }
}
catch {
    Write-Host "  ⚠ Erreur de test réseau: $_" -ForegroundColor Yellow
}

# ========== CONNEXION HMAILSERVER ==========
Write-Host "`n[1/4] Connexion à hMailServer..." -ForegroundColor Yellow

$hMailApp = $null
$domain = $null

try {
    Write-Host "  → Création de l'objet COM hMailServer.Application..." -ForegroundColor Gray
    $hMailApp = New-Object -ComObject hMailServer.Application
    
    if ($hMailApp -eq $null) {
        throw "L'objet hMailServer.Application est null"
    }
    
    Write-Host "  → Authentification..." -ForegroundColor Gray
    $authResult = $hMailApp.Authenticate("Administrator", $hMailAdminPassword)
    
    if (-not $authResult) {
        throw "Échec de l'authentification hMailServer. Vérifiez le mot de passe."
    }
    
    Write-Host "  → Récupération du domaine $defaultDomain..." -ForegroundColor Gray
    $domain = $hMailApp.Domains.ItemByName($defaultDomain)
    
    if ($domain -eq $null) {
        Write-Host "`n✗ Domaine '$defaultDomain' non trouvé!" -ForegroundColor Red
        Write-Host "`nDomaines disponibles:" -ForegroundColor Yellow
        for ($i = 0; $i -lt $hMailApp.Domains.Count; $i++) {
            $d = $hMailApp.Domains.Item($i)
            Write-Host "  - $($d.Name)" -ForegroundColor Cyan
        }
        exit
    }
    
    Write-Host "✓ Connecté à hMailServer - Domaine: $defaultDomain" -ForegroundColor Green
}
catch {
    Write-Host "`n✗ ERREUR de connexion hMailServer:" -ForegroundColor Red
    Write-Host $_.Exception.Message -ForegroundColor Red
    Write-Host "`nVérifications:" -ForegroundColor Yellow
    Write-Host "1. Service hMailServer démarré ?" -ForegroundColor White
    Write-Host "2. PowerShell exécuté en Administrateur ?" -ForegroundColor White
    Write-Host "3. Mot de passe administrateur correct ?" -ForegroundColor White
    exit
}

# ========== RÉCUPÉRATION DES COMPTES HMAILSERVER ==========
Write-Host "`n[2/4] Récupération des comptes hMailServer existants..." -ForegroundColor Yellow

$hmailAccounts = @{}
try {
    for ($i = 0; $i -lt $domain.Accounts.Count; $i++) {
        $account = $domain.Accounts.Item($i)
        $hmailAccounts[$account.Address.ToLower()] = $account
    }
    Write-Host "✓ $($hmailAccounts.Count) comptes trouvés" -ForegroundColor Green
}
catch {
    Write-Host "✗ Erreur: $_" -ForegroundColor Red
    exit
}

# ========== CONNEXION À L'AD DISTANT ==========
Write-Host "`n[3/4] Connexion à l'Active Directory distant..." -ForegroundColor Yellow
Write-Host "  Serveur: $adServer" -ForegroundColor Gray
Write-Host "  Chemin LDAP: $ldapPath" -ForegroundColor Gray

try {
    # Créer des credentials pour l'AD distant
    $secPassword = ConvertTo-SecureString $adPassword -AsPlainText -Force
    $credential = New-Object System.Management.Automation.PSCredential($adUsername, $secPassword)
    
    # Créer la connexion LDAP avec authentification
    Write-Host "  → Authentification sur l'AD..." -ForegroundColor Gray
    
    $directoryEntry = New-Object System.DirectoryServices.DirectoryEntry(
        $ldapPath,
        $adUsername,
        $adPassword,
        [System.DirectoryServices.AuthenticationTypes]::Secure
    )
    
    # Test de la connexion
    $testBind = $directoryEntry.Name
    if ([string]::IsNullOrEmpty($testBind) -and $directoryEntry.NativeObject -eq $null) {
        throw "Impossible de se connecter à l'AD. Vérifiez les credentials et le chemin LDAP."
    }
    
    Write-Host "  ✓ Authentifié sur l'AD" -ForegroundColor Green
    
    # Créer le searcher
    Write-Host "  → Recherche des utilisateurs..." -ForegroundColor Gray
    $searcher = New-Object System.DirectoryServices.DirectorySearcher
    $searcher.SearchRoot = $directoryEntry
    
    # Filtre pour les utilisateurs avec email
    $searcher.Filter = "(&(objectClass=user)(objectCategory=person)(mail=*))"
    
    # Propriétés à récupérer
    $searcher.PropertiesToLoad.AddRange(@(
        "sAMAccountName",
        "mail",
        "givenName",
        "sn",
        "displayName",
        "userAccountControl"
    ))
    
    $searcher.PageSize = 1000
    $searcher.SearchScope = [System.DirectoryServices.SearchScope]::Subtree
    
    $results = $searcher.FindAll()
    
    Write-Host "✓ $($results.Count) utilisateurs trouvés dans AD" -ForegroundColor Green
}
catch {
    Write-Host "`n✗ ERREUR de connexion AD:" -ForegroundColor Red
    Write-Host $_.Exception.Message -ForegroundColor Red
    Write-Host "`nVérifications:" -ForegroundColor Yellow
    Write-Host "1. Le serveur AD '$adServer' est-il accessible ?" -ForegroundColor White
    Write-Host "2. Les credentials AD sont-ils corrects ?" -ForegroundColor White
    Write-Host "   Username: $adUsername" -ForegroundColor Gray
    Write-Host "3. Le chemin LDAP est-il correct ?" -ForegroundColor White
    Write-Host "   $ldapPath" -ForegroundColor Gray
    Write-Host "4. Le port 389 (LDAP) est-il ouvert ?" -ForegroundColor White
    Write-Host "5. Essayez avec LDAPS (port 636) si disponible" -ForegroundColor White
    exit
}

# ========== SYNCHRONISATION ==========
Write-Host "`n[4/4] Synchronisation en cours..." -ForegroundColor Yellow
Write-Host "────────────────────────────────────────" -ForegroundColor Gray

$stats = @{
    Created = 0
    Updated = 0
    Skipped = 0
    Errors = 0
}

foreach ($result in $results) {
    $props = $result.Properties
    
    # Récupérer les informations
    $samAccount = if ($props["samaccountname"].Count -gt 0) { $props["samaccountname"][0] } else { "Inconnu" }
    $email = if ($props["mail"].Count -gt 0) { $props["mail"][0] } else { $null }
    $firstName = if ($props["givenname"].Count -gt 0) { $props["givenname"][0] } else { "" }
    $lastName = if ($props["sn"].Count -gt 0) { $props["sn"][0] } else { "" }
    $displayName = if ($props["displayname"].Count -gt 0) { $props["displayname"][0] } else { $samAccount }
    
    # Vérifier si le compte est actif dans AD
    $userAccountControl = if ($props["useraccountcontrol"].Count -gt 0) { $props["useraccountcontrol"][0] } else { 0 }
    $isEnabled = -not ($userAccountControl -band 0x2)  # 0x2 = ACCOUNTDISABLE
    
    # Vérifier l'adresse email
    if ([string]::IsNullOrWhiteSpace($email)) {
        Write-Host "⊘ $displayName - Pas d'email" -ForegroundColor DarkGray
        $stats.Skipped++
        continue
    }
    
    # Forcer le domaine mail si l'email AD ne correspond pas
    if (-not $email.EndsWith("@$defaultDomain")) {
        $emailParts = $email.Split('@')
        $email = "$($emailParts[0])@$defaultDomain"
        Write-Host "ℹ $displayName - Email modifié: $email" -ForegroundColor Cyan
    }
    
    $emailLower = $email.ToLower()
    
    try {
        # Vérifier si le compte existe dans hMailServer
        if ($hmailAccounts.ContainsKey($emailLower)) {
            # MISE À JOUR
            if ($updateExistingAccounts) {
                $account = $hmailAccounts[$emailLower]
                
                $account.PersonFirstName = $firstName
                $account.PersonLastName = $lastName
                $account.Active = $isEnabled
                $account.Save()
                
                Write-Host "↻ $email - Mis à jour" -ForegroundColor Cyan
                $stats.Updated++
            }
            else {
                Write-Host "→ $email - Existe déjà" -ForegroundColor Gray
                $stats.Skipped++
            }
        }
        else {
            # CRÉATION
            if ($createNewAccounts) {
                $newAccount = $domain.Accounts.Add()
                $newAccount.Address = $email
                $newAccount.Password = $defaultPassword
                $newAccount.Active = if ($enableAccountsByDefault) { $isEnabled } else { $false }
                $newAccount.PersonFirstName = $firstName
                $newAccount.PersonLastName = $lastName
                $newAccount.MaxSize = $accountMaxSize
                $newAccount.Save()
                
                Write-Host "✓ $email - Créé" -ForegroundColor Green
                $stats.Created++
            }
            else {
                Write-Host "⊘ $email - Nouveau (création désactivée)" -ForegroundColor DarkGray
                $stats.Skipped++
            }
        }
    }
    catch {
        Write-Host "✗ $email - ERREUR: $_" -ForegroundColor Red
        $stats.Errors++
    }
}

# ========== RÉSULTATS ==========
Write-Host "`n========================================" -ForegroundColor Cyan
Write-Host "RÉSUMÉ DE LA SYNCHRONISATION" -ForegroundColor Cyan
Write-Host "========================================" -ForegroundColor Cyan
Write-Host "Comptes créés    : $($stats.Created)" -ForegroundColor Green
Write-Host "Comptes mis à jour: $($stats.Updated)" -ForegroundColor Cyan
Write-Host "Comptes ignorés  : $($stats.Skipped)" -ForegroundColor Gray
Write-Host "Erreurs          : $($stats.Errors)" -ForegroundColor Red
Write-Host "────────────────────────────────────────" -ForegroundColor Gray
Write-Host "Total hMailServer: $($domain.Accounts.Count) comptes" -ForegroundColor White
Write-Host "========================================`n" -ForegroundColor Cyan

# ========== NETTOYAGE ==========
if ($results) { $results.Dispose() }
if ($searcher) { $searcher.Dispose() }
if ($directoryEntry) { $directoryEntry.Dispose() }
if ($domain -ne $null) {
    [System.Runtime.Interopservices.Marshal]::ReleaseComObject($domain) | Out-Null
}
if ($hMailApp -ne $null) {
    [System.Runtime.Interopservices.Marshal]::ReleaseComObject($hMailApp) | Out-Null
}
[System.GC]::Collect()
[System.GC]::WaitForPendingFinalizers()

Write-Host "Synchronisation terminée!" -ForegroundColor Green
``` 


---

---

## 6. Créer un compte de messagerie Thunderbird 

    Entrer votre nom complet: Jean
    Adresse e-mail : jean@orleans.sportludique.fr
    Mot de passe : *********
#### Paramètre du serveur 
```
Serveur entrant 
Protocole : IMAP
Nom d'hôte : imap.orleans.sportludique.fr
port: non sécurisé 25 ou sécurisé 587
Sécurité de la connection : Aucun si port 25 | STARTTLS si port 587
Méthode d'authentification: 
```

```
Serveur sortant
Nom d'hôte : smtp.orleans.sportludique.fr
port: 143 ou 993
Sécurité de la connection : Aucun si port 25 | STARTTLS si port 587
Méthode d'authentification: 
```


---

---

 ## 7. routeur


``` 
redirection de port 25 587
``` 

``` 

``` 
---

---