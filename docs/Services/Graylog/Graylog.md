# Installation Graylog sur Debian 12

---

## Prérequis

Avant de commencer, assurez-vous que :

*   Tous les **ports requis sont ouverts** entre les nœuds.
*   Les **pare-feu sont désactivés** ou configurés pour autoriser le trafic.
*   Un **load balancer** avec DNS est configuré pour Graylog (recommandé).
*   Le système de fichiers **XFS** est utilisé pour le stockage (recommandé).

### Ports requis

| Service | Port | Description |
|---------|------|-------------|
| Graylog Web/API | 9000 | Interface web et API REST |
| MongoDB | 27017 | Base de données |
| Data Node | 9200 | OpenSearch API |
| Data Node | 9300 | Communication inter-nœuds |
| Graylog Inputs | 1514, 514 | Réception des logs |

---

---

## Configuration Système

### Configuration du fuseau horaire

!!! info "Timezone"
    Définir le fuseau horaire sur tous les serveurs :
    ```bash
    sudo timedatectl set-timezone Europe/Paris
    ```
    
    Vérification :
    ```bash
    timedatectl
    ```

---

---

## Étape 1 : Installation de MongoDB

MongoDB sert de base de données de métadonnées. À installer sur tous les nœuds Graylog/MongoDB.

### 1.1 Installation

!!! info "Commandes d'installation"
    Importer la clé GPG :
    ```bash
    curl -fsSL https://www.mongodb.org/static/pgp/server-8.0.asc | \
      sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg --dearmor
    ```

    Ajouter le dépôt :
    ```bash
    echo "deb [ signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] http://repo.mongodb.org/apt/debian bookworm/mongodb-org/8.0 main" | \
      sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
    ```

    Installer :
    ```bash
    sudo apt-get update
    sudo apt-get install -y mongodb-org
    sudo apt-mark hold mongodb-org
    ```

### 1.2 Configuration

!!! info
    `sudo nano /etc/mongod.conf`

    Modifier la section `net:` :
    ```yaml
    net:
      port: 27017
      bindIp: 0.0.0.0  # Ou IP spécifique
    ```

### 1.3 Démarrage

!!! info
    ```bash
    sudo systemctl daemon-reload
    sudo systemctl enable mongod.service
    sudo systemctl start mongod.service
    ```

---

---

## Étape 2 : Installation de Data Node (OpenSearch)

Data Node gère l'ingestion et l'indexation.

### 2.1 Installation

!!! info
    ```bash
    wget https://packages.graylog2.org/repo/packages/graylog-7.0-repository_latest.deb
    sudo dpkg -i graylog-7.0-repository_latest.deb
    sudo apt-get update
    sudo apt-get install graylog-datanode
    ```

### 2.2 Configuration Système (vm.max_map_count)

!!! info
    Augmenter la limite mémoire :
    ```bash
    echo 'vm.max_map_count=262144' | sudo tee -a /etc/sysctl.d/99-graylog-datanode.conf
    sudo sysctl --system
    ```

### 2.3 Configuration Data Node

Générer le `password_secret` (à conserver !) :

!!! info "Génération Secret"
    ```bash
    openssl rand -hex 32
    ```

Éditer la configuration :

!!! info
    `sudo nano /etc/graylog/datanode/datanode.conf`

    Paramètres clés :
    ```properties
    # Secret généré (MEME secret sur TOUS les noeuds)
    password_secret = c0b10ff32ccd565dd76a6331aeba6ff9bce2e6f1c8ca...
    
    # Heap Size (50% RAM, max 31g)
    opensearch_heap = 4g
    
    # Connexion MongoDB
    mongodb_uri = mongodb://192.168.140.235:27017/graylog?replicaSet=rs0
    ```

### 2.4 Démarrage

!!! info
    ```bash
    sudo systemctl daemon-reload
    sudo systemctl enable graylog-datanode
    sudo systemctl start graylog-datanode
    ```

---

---

## Étape 3 : Installation de Graylog Server

### 3.1 Installation

!!! info
    ```bash
    sudo apt-get install graylog-server
    ```

### 3.2 Configuration

Générer le mot de passe root (SHA256) :

!!! info
    ```bash
    echo -n "VotreMotDePasse" | sha256sum | cut -d' ' -f1
    ```

Éditer le configuration :

!!! info
    `sudo nano /etc/graylog/server/server.conf`

    Paramètres clés :
    ```properties
    # Hash du mot de passe admin
    root_password_sha2 = 86dac5b7dc404d676e3696eb29337345...
    
    # MEME secret que Data Node
    password_secret = c0b10ff32ccd565dd76a6331aeba6ff9bce2e6f1c8ca...
    
    # Ecoute Web
    http_bind_address = 0.0.0.0:9000
    
    # Connexion MongoDB
    mongodb_uri = mongodb://192.168.140.235:27017/graylog
    
    # Rétention journal
    message_journal_max_age = 72h
    message_journal_max_size = 4gb
    ```

### 3.3 Configuration Java Heap

!!! info
    `sudo nano /etc/default/graylog-server`

    Ajuster la mémoire (50% RAM, max 16GB) :
    ```bash
    GRAYLOG_SERVER_JAVA_OPTS="-Xms2g -Xmx2g -server -XX:+UseG1GC ..."
    ```

### 3.4 Démarrage

!!! info
    ```bash
    sudo systemctl daemon-reload
    sudo systemctl enable graylog-server
    sudo systemctl start graylog-server
    ```

---

---

## Étape 4 : Premier Démarrage (Preflight)

### 4.1 Identifiants temporaires

Si le serveur démarre en mode "Preflight" (première install), trouver les identifiants dans les logs :

!!! info
    ```bash
    sudo journalctl -u graylog-server -n 100
    ```
    
    Chercher : `Initial configuration is accessible at...`

### 4.2 Accès Web

Accédez à `http://IP_GRAYLOG:9000`.

*   Utilisez `admin` et le mot de passe temporaire ou celui défini si l'installation est complète.

### 4.3 Configuration Hosts

Pour que Graylog et DataNode communiquent correctement, assurez-vous que les `hostname` sont résolus dans `/etc/hosts`.

!!! info
    `sudo nano /etc/hosts`

    ```text
    127.0.0.1       localhost
    192.168.140.235 ORL-SRV-Graylog
    192.168.140.10  Datanode
    ```

---
Pour la configuration de l'interface web, voir : [Guide Web Graylog](Graylog-web.md).
