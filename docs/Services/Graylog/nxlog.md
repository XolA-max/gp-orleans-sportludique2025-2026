# Envoyer les logs Windows vers Graylog avec NXLog

---

## Présentation

Ce tutoriel explique comment installer et configurer **NXLog** sur Windows Server pour envoyer les journaux Windows (Event Viewer) vers un serveur **Graylog**.

---

## 1. Créer un Input GELF UDP dans Graylog

Avant de configurer Windows, Graylog doit être prêt à recevoir les messages.

1.  Connectez-vous à l'interface web de Graylog.
2.  Aller dans **System** → **Inputs**.
3.  Sélectionner **GELF UDP**.
4.  Cliquer sur **Launch new input**.

### Configuration

!!! info "Paramètres Input"
    *   **Title** : `Windows GELF UDP`
    *   **Bind address** : `0.0.0.0`
    *   **Port** : `12201` (Port par défaut GELF)

---

---

## 2. Installer et Configurer NXLog sur Windows

### 2.1 Installation

1.  Téléchargez **NXLog Community Edition** (Windows x64).
2.  Installez le package MSI (`nxlog-ce-x.x.x.msi`).

### 2.2 Configuration

Éditez le fichier de configuration : `C:\Program Files\nxlog\conf\nxlog.conf`

!!! info "nxlog.conf"
    Ajouter ou modifier le contenu suivant :
    
    ```conf
    ## Extension GELF pour le formatage
    <Extension gelf>
        Module      xm_gelf
    </Extension>
    
    ## Input pour lire les logs Windows
    <Input in>
        Module      im_msvistalog
        # Vous pouvez filtrer ici si besoin
    </Input>
    
    ## Output vers Graylog
    <Output graylog_udp>
        Module      om_udp
        Host        192.168.10.220
        Port        12201
        OutputType  GELF_UDP
    </Output>
    
    ## Route
    <Route 1>
        Path        in => graylog_udp
    </Route>
    ```

    *Remplacer `192.168.10.220` par l'IP de votre serveur Graylog.*

### 2.3 Redémarrage du service

Pour appliquer la configuration, redémarrez le service NXLog.

!!! info "PowerShell (Admin)"
    ```powershell
    Restart-Service nxlog
    ```

### 2.4 Vérification

En cas de problème, vérifiez le fichier de log local : `C:\Program Files\nxlog\data\nxlog.log`.

---

---

## 3. Vérification dans Graylog

1.  Aller dans **Search**.
2.  Sélectionner la période **Last 5 minutes**.
3.  Vous devriez voir arriver les logs Windows.
