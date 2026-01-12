# Inventaire GLPI - Windows

Script PowerShell pour installer un agent et forcer l'inventaire dans GLPI.

---

## Script Auto-Install

Ce script télécharge, installe et configure l'agent GLPI sur Windows Serveur.

!!! info "Prérequis"
    *   Exécuter en tant qu'Administrateur
    *   Serveur GLPI accessible : `http://192.168.140.225/front/inventory.php`

```powershell
#Requires -RunAsAdministrator

<#
.SYNOPSIS
    Script de déploiement automatique de l'agent GLPI
.DESCRIPTION
    Ce script télécharge, installe et configure l'agent GLPI sur Windows
    Serveur GLPI configuré : http://192.168.140.225/front/inventory.php
.PARAMETER Tag
    Tag pour identifier le groupe de machines (par défaut: Production)
.PARAMETER AgentVersion
    Version de l'agent à installer (par défaut: 1.11)
.PARAMETER ServerURL
    URL du serveur GLPI (par défaut: http://192.168.140.225/front/inventory.php)
#>

param(
    [Parameter(Mandatory=$false)]
    [string]$ServerURL = "http://192.168.140.225/front/inventory.php",
    
    [Parameter(Mandatory=$false)]
    [string]$Tag = "Production",
    
    [Parameter(Mandatory=$false)]
    [string]$AgentVersion = "1.11",
    
    [Parameter(Mandatory=$false)]
    [string]$DownloadPath = "$env:TEMP\GLPI-Agent",
    
    [Parameter(Mandatory=$false)]
    [switch]$Force
)

# Configuration
$ErrorActionPreference = "Stop"
$ProgressPreference = 'SilentlyContinue'

# Détection de l'architecture
$Architecture = if ([Environment]::Is64BitOperatingSystem) { "x64" } else { "x86" }

# URL de téléchargement (GitHub Releases)
$AgentFileName = "GLPI-Agent-${AgentVersion}-${Architecture}.msi"
$DownloadURL = "https://github.com/glpi-project/glpi-agent/releases/download/${AgentVersion}/${AgentFileName}"
$InstallerPath = Join-Path $DownloadPath $AgentFileName

# Fonction de logging
function Write-Log {
    param([string]$Message, [string]$Level = "INFO")
    $Timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    $LogMessage = "[$Timestamp] [$Level] $Message"
    
    # Couleurs selon le niveau
    switch ($Level) {
        "ERROR"   { Write-Host $LogMessage -ForegroundColor Red }
        "WARNING" { Write-Host $LogMessage -ForegroundColor Yellow }
        "SUCCESS" { Write-Host $LogMessage -ForegroundColor Green }
        default   { Write-Host $LogMessage -ForegroundColor White }
    }
    
    Add-Content -Path "$DownloadPath\glpi-deployment.log" -Value $LogMessage -ErrorAction SilentlyContinue
}

# Fonction pour vérifier si l'agent est déjà installé
function Test-GLPIAgentInstalled {
    $InstalledAgent = Get-WmiObject -Class Win32_Product -Filter "Name LIKE '%GLPI%Agent%'" -ErrorAction SilentlyContinue
    if ($InstalledAgent) {
        Write-Log "Agent GLPI déjà installé: $($InstalledAgent.Name) - Version: $($InstalledAgent.Version)" "WARNING"
        return $true
    }
    return $false
}

# Fonction pour désinstaller l'agent existant
function Uninstall-GLPIAgent {
    Write-Log "Désinstallation de l'agent GLPI existant..."
    $InstalledAgent = Get-WmiObject -Class Win32_Product -Filter "Name LIKE '%GLPI%Agent%'"
    if ($InstalledAgent) {
        $InstalledAgent.Uninstall() | Out-Null
        Write-Log "Agent désinstallé avec succès" "SUCCESS"
        Start-Sleep -Seconds 5
    }
}

# Fonction de vérification de connectivité
function Test-GLPIServerConnectivity {
    param([string]$ServerURL)
    
    Write-Log "Vérification de la connectivité avec le serveur GLPI..."
    
    try {
        # Extraire l'hôte et le port de l'URL
        $Uri = [System.Uri]$ServerURL
        $HostName = $Uri.Host
        $Port = if ($Uri.Port -eq -1) { 80 } else { $Uri.Port }
        
        # Test de connectivité TCP
        $TcpTest = Test-NetConnection -ComputerName $HostName -Port $Port -WarningAction SilentlyContinue
        
        if ($TcpTest.TcpTestSucceeded) {
            Write-Log "Connexion au serveur GLPI réussie ($HostName`:$Port)" "SUCCESS"
            return $true
        }
        else {
            Write-Log "Impossible de joindre le serveur GLPI ($HostName`:$Port)" "ERROR"
            return $false
        }
    }
    catch {
        Write-Log "Erreur lors du test de connectivité: $_" "ERROR"
        return $false
    }
}

# Fonction de téléchargement
function Download-GLPIAgent {
    param([string]$URL, [string]$Destination)
    
    Write-Log "Téléchargement de l'agent GLPI depuis: $URL"
    
    try {
        # Créer le dossier de destination s'il n'existe pas
        if (-not (Test-Path $DownloadPath)) {
            New-Item -ItemType Directory -Path $DownloadPath -Force | Out-Null
        }
        
        # Téléchargement avec WebClient
        $WebClient = New-Object System.Net.WebClient
        $WebClient.DownloadFile($URL, $Destination)
        
        # Vérifier que le fichier a été téléchargé
        if (Test-Path $Destination) {
            $FileSize = (Get-Item $Destination).Length / 1MB
            Write-Log "Téléchargement terminé: $Destination ($('{0:N2}' -f $FileSize) MB)" "SUCCESS"
            return $true
        }
        else {
            Write-Log "Le fichier n'a pas été téléchargé correctement" "ERROR"
            return $false
        }
    }
    catch {
        Write-Log "Erreur lors du téléchargement: $_" "ERROR"
        return $false
    }
}

# Fonction d'installation
function Install-GLPIAgent {
    param([string]$InstallerPath, [string]$Server, [string]$Tag)
    
    Write-Log "Installation de l'agent GLPI..."
    Write-Log "Serveur cible: $Server"
    Write-Log "Tag: $Tag"
    
    # Paramètres d'installation
    $InstallArgs = @(
        "/i"
        "`"$InstallerPath`""
        "/quiet"
        "/norestart"
        "SERVER=`"$Server`""
        "TAG=`"$Tag`""
        "RUNNOW=1"
        "ADD_FIREWALL_EXCEPTION=1"
        "EXECMODE=2"  # Mode service
        "/L*V"
        "`"$DownloadPath\glpi-install.log`""
    )
    
    try {
        Write-Log "Lancement de l'installation MSI..."
        $Process = Start-Process -FilePath "msiexec.exe" -ArgumentList $InstallArgs -Wait -PassThru
        
        if ($Process.ExitCode -eq 0) {
            Write-Log "Installation réussie (Code: 0)" "SUCCESS"
            return $true
        }
        elseif ($Process.ExitCode -eq 3010) {
            Write-Log "Installation réussie - Redémarrage requis (Code: 3010)" "WARNING"
            return $true
        }
        else {
            Write-Log "Erreur d'installation. Code de sortie: $($Process.ExitCode)" "ERROR"
            Write-Log "Consultez le log: $DownloadPath\glpi-install.log" "ERROR"
            return $false
        }
    }
    catch {
        Write-Log "Erreur lors de l'installation: $_" "ERROR"
        return $false
    }
}

# Fonction de vérification du service
function Test-GLPIService {
    Write-Log "Vérification du service GLPI Agent..."
    
    Start-Sleep -Seconds 3
    $Service = Get-Service -Name "GLPI-Agent" -ErrorAction SilentlyContinue
    
    if ($Service) {
        Write-Log "Service trouvé - État: $($Service.Status)" "SUCCESS"
        
        if ($Service.Status -ne "Running") {
            Write-Log "Démarrage du service..."
            Start-Service -Name "GLPI-Agent"
            Start-Sleep -Seconds 3
            $Service.Refresh()
            Write-Log "Service démarré - État: $($Service.Status)" "SUCCESS"
        }
        return $true
    }
    else {
        Write-Log "Service GLPI Agent non trouvé" "ERROR"
        return $false
    }
}

# Fonction pour forcer l'inventaire
function Invoke-GLPIInventory {
    Write-Log "Exécution d'un inventaire immédiat..."
    
    $AgentExe = "C:\Program Files\GLPI-Agent\glpi-agent.bat"
    
    if (Test-Path $AgentExe) {
        try {
            $Arguments = "--server `"$ServerURL`" --logger=stderr"
            Write-Log "Commande: $AgentExe $Arguments"
            Start-Process -FilePath $AgentExe -ArgumentList $Arguments -NoNewWindow -Wait
            Write-Log "Inventaire exécuté avec succès" "SUCCESS"
        }
        catch {
            Write-Log "Erreur lors de l'exécution de l'inventaire: $_" "WARNING"
        }
    }
    else {
        Write-Log "Fichier glpi-agent.bat non trouvé" "WARNING"
    }
}

# Fonction pour afficher la configuration
function Show-GLPIConfiguration {
    $ConfigFile = "C:\Program Files\GLPI-Agent\etc\agent.cfg"
    
    if (Test-Path $ConfigFile) {
        Write-Log "`nConfiguration actuelle:"
        $Config = Get-Content $ConfigFile
        foreach ($Line in $Config) {
            if ($Line -match "^server|^tag|^delaytime") {
                Write-Log "  $Line"
            }
        }
    }
}

# Fonction de nettoyage
function Clear-TemporaryFiles {
    param([bool]$KeepLogs = $true)
    
    Write-Log "Nettoyage des fichiers temporaires..."
    
    if (Test-Path $InstallerPath) {
        Remove-Item -Path $InstallerPath -Force -ErrorAction SilentlyContinue
        Write-Log "Installeur supprimé" "SUCCESS"
    }
    
    if (-not $KeepLogs) {
        Remove-Item -Path "$DownloadPath\*.log" -Force -ErrorAction SilentlyContinue
    }
}

# ===== SCRIPT PRINCIPAL =====

Write-Host "`n" -NoNewline
Write-Host "╔════════════════════════════════════════════════════════════╗" -ForegroundColor Cyan
Write-Host "║       DÉPLOIEMENT AGENT GLPI - Version $AgentVersion            ║" -ForegroundColor Cyan
Write-Host "╚════════════════════════════════════════════════════════════╝" -ForegroundColor Cyan
Write-Host ""

Write-Log "===== DÉBUT DU DÉPLOIEMENT GLPI AGENT ====="
Write-Log "Système: Windows $([Environment]::OSVersion.Version) - Architecture: $Architecture"
Write-Log "Serveur GLPI: $ServerURL"
Write-Log "Tag: $Tag"
Write-Log "Version de l'agent: $AgentVersion"
Write-Log "Dossier de travail: $DownloadPath"

# Créer le dossier de logs
if (-not (Test-Path $DownloadPath)) {
    New-Item -ItemType Directory -Path $DownloadPath -Force | Out-Null
}

# Vérification des droits administrateur
if (-not ([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)) {
    Write-Log "Ce script nécessite des droits administrateur!" "ERROR"
    Write-Host "`nAppuyez sur une touche pour quitter..." -ForegroundColor Yellow
    if ($Host.UI.RawUI) {
        $null = $Host.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown")
    }
    exit 1
}

# Étape 0: Vérifier la connectivité au serveur GLPI
if (-not (Test-GLPIServerConnectivity -ServerURL $ServerURL)) {
    Write-Log "Le serveur GLPI n'est pas accessible. Vérifiez l'adresse et votre connexion réseau." "ERROR"
    Write-Host "`nAppuyez sur une touche pour quitter..." -ForegroundColor Yellow
    if ($Host.UI.RawUI) {
        $null = $Host.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown")
    }
    exit 1
}

# Étape 1: Vérifier si déjà installé
if ((Test-GLPIAgentInstalled) -and -not $Force) {
    Write-Log "L'agent est déjà installé. Utilisez -Force pour réinstaller." "WARNING"
    Write-Host "`nAppuyez sur une touche pour quitter..." -ForegroundColor Yellow
    if ($Host.UI.RawUI) {
        $null = $Host.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown")
    }
    exit 0
}
elseif ((Test-GLPIAgentInstalled) -and $Force) {
    Uninstall-GLPIAgent
}

# Étape 2: Téléchargement
if (-not (Test-Path $InstallerPath)) {
    if (-not (Download-GLPIAgent -URL $DownloadURL -Destination $InstallerPath)) {
        Write-Log "Échec du téléchargement. Arrêt du script." "ERROR"
        Write-Host "`nAppuyez sur une touche pour quitter..." -ForegroundColor Yellow
        if ($Host.UI.RawUI) {
            $null = $Host.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown")
        }
        exit 1
    }
}
else {
    Write-Log "Fichier d'installation déjà présent: $InstallerPath"
}

# Étape 3: Installation
if (-not (Install-GLPIAgent -InstallerPath $InstallerPath -Server $ServerURL -Tag $Tag)) {
    Write-Log "Échec de l'installation. Arrêt du script." "ERROR"
    Write-Host "`nAppuyez sur une touche pour quitter..." -ForegroundColor Yellow
    if ($Host.UI.RawUI) {
        $null = $Host.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown")
    }
    exit 1
}

# Étape 4: Vérification du service
if (-not (Test-GLPIService)) {
    Write-Log "Le service n'a pas pu être vérifié." "WARNING"
}

# Étape 5: Afficher la configuration
Show-GLPIConfiguration

# Étape 6: Exécution d'un inventaire
Invoke-GLPIInventory

# Étape 7: Nettoyage
Clear-TemporaryFiles -KeepLogs $true

Write-Log "===== DÉPLOIEMENT TERMINÉ AVEC SUCCÈS ====="

# Afficher les informations finales
Write-Host "`n╔════════════════════════════════════════════════════════════╗" -ForegroundColor Green
Write-Host "║           DÉPLOIEMENT TERMINÉ AVEC SUCCÈS                  ║" -ForegroundColor Green
Write-Host "╚════════════════════════════════════════════════════════════╝" -ForegroundColor Green
Write-Host ""
Write-Host "  📡 Serveur GLPI : " -NoNewline -ForegroundColor White
Write-Host "$ServerURL" -ForegroundColor Cyan
Write-Host "  🏷️  Tag          : " -NoNewline -ForegroundColor White
Write-Host "$Tag" -ForegroundColor Cyan
Write-Host "  📝 Logs         : " -NoNewline -ForegroundColor White
Write-Host "$DownloadPath\glpi-deployment.log" -ForegroundColor Cyan
Write-Host "  ⚙️  Configuration: " -NoNewline -ForegroundColor White
Write-Host "C:\Program Files\GLPI-Agent\etc\agent.cfg" -ForegroundColor Cyan
Write-Host ""
Write-Host "  ✅ L'inventaire a été envoyé au serveur GLPI" -ForegroundColor Green
Write-Host "  ✅ Vérifiez dans GLPI : Parc > Ordinateurs" -ForegroundColor Green
Write-Host ""

Write-Host "Appuyez sur une touche pour quitter..." -ForegroundColor Yellow
if ($Host.UI.RawUI) {
    $null = $Host.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown")
}
```