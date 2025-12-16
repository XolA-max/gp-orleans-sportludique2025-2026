# Envoyer les logs Windows vers Graylog avec NXLog

## Présentation

Ce tutoriel explique comment installer et configurer **NXLog** sur Windows Server pour envoyer les journaux Windows vers un serveur **Graylog**. Cette solution permet de centraliser et d'indexer les journaux de l'Observateur d'événements de plusieurs machines Windows dans Graylog.

---

## Créer un input NXLog dans Graylog

### Étape 1 : Accéder aux Inputs

1. Connectez-vous à l'interface web de Graylog
2. Cliquez sur le menu **"Système"**
3. Cliquez sur **"Inputs"**
4. Sélectionnez **"GELF UDP"** dans la liste
5. Cliquez sur **"Launch new input"**

### Étape 2 : Configurer l'Input

Paramètres à configurer :

- **Nom** : `Graylog_UDP_NXLogs_Windows` (ou un nom de votre choix)
- **Bind address** : `0.0.0.0` (pour être accessible sur toutes les interfaces)
- **Port** : `12201` (port par défaut)

Validez la configuration. L'Input est maintenant prêt à recevoir des logs.

---

## Installer et configurer NXLog sur Windows

### A. Installer NXLog sur Windows Server

1. Téléchargez **NXLog Community Edition** depuis le site officiel
2. Sélectionnez la version **Windows x86-64**
3. Lancez l'installation du package MSI `nxlog-ce-3.2.2329.msi`
4. Suivez l'assistant d'installation

### B. Configurer NXLog pour Graylog

#### Configuration de base

Éditez le fichier de configuration situé à :
```
C:\Program Files\nxlog\conf\nxlog.conf
```

Ajoutez les lignes suivantes à la fin du fichier :
```conf
# Récupérer les journaux de l'observateur d'événements
<Input in>
    Module      im_msvistalog
</Input>

# Déclarer le serveur Graylog (selon input)
<Extension gelf>
    Module        xm_gelf
</Extension>

<Output graylog_udp>
    Module        om_udp
    Host          192.168.10.220
    Port          12201
    OutputType    GELF_UDP
</o>

# Routage des flux in vers out
<Route 1>
    Path        in => graylog_udp
</Route>
```

#### Redémarrer le service

Ouvrez PowerShell en tant qu'administrateur et exécutez :
```powershell
Restart-Service nxlog
```

#### Fichier de log NXLog

En cas de problème, consultez le fichier de log :
```
C:\Program Files\nxlog\data\nxlog.log
```
## Recevoir les journaux Windows dans Graylog

### Vérifier la réception des logs

1. Cliquez sur **"Search"** dans le menu de Graylog
2. Vous devriez voir un pic de logs correspondant à l'arrivée des premiers journaux
3. Activez le rafraîchissement automatique (toutes les 5 secondes) pour voir les logs en temps réel

### Consulter un log

Cliquez sur un log dans la liste pour visualiser son contenu détaillé (équivalent à l'Observateur d'événements de Windows).

