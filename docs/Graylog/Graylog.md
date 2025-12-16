# Installation Graylog Multi-Node sur Debian 12


##  Prérequis

Avant de commencer, assurez-vous que :

- Tous les **ports requis sont ouverts** entre les nœuds
- Les **pare-feu sont désactivés** ou configurés pour autoriser le trafic
-  Un **load balancer** avec DNS est configuré pour Graylog (recommandé)
-  Le système de fichiers **XFS** est utilisé pour le stockage (fortement recommandé)
-  Vous avez vérifié les **ressources recommandées** selon l'architecture Conventionnelle

### Ports par défaut requis

| Service | Port | Description |
|---------|------|-------------|
| Graylog Web/API | 9000 | Interface web et API REST |
| MongoDB | 27017 | Base de données |
| Data Node | 9200 | OpenSearch API |
| Data Node | 9300 | Communication inter-nœuds |
| Graylog Inputs | 1514, 5044, etc. | Réception des logs |

---

## Configuration du fuseau horaire

Définir le fuseau horaire sur tous les serveurs :

```bash
sudo timedatectl set-timezone UTC
```

Ou pour l'Europe :

```bash
sudo timedatectl set-timezone Europe/Paris
```

**Vérification :**
```bash
timedatectl
```

---

##  Fichiers de configuration importants

| Service | Fichier de configuration | Description |
|---------|-------------------------|-------------|
| **MongoDB** | `/etc/mongod.conf` | Configuration MongoDB |
| **Data Node** | `/etc/graylog/datanode/datanode.conf` | Configuration Data Node |
| **Graylog** | `/etc/graylog/server/server.conf` | Configuration Graylog |
| **Graylog JVM** | `/etc/default/graylog-server` | Paramètres heap Java |

---

## 🗄️ Étape 1 : Installation de MongoDB

MongoDB sert de base de données de métadonnées pour Graylog. Vous devez l'installer sur **tous les nœuds Graylog/MongoDB**.

> **Ressources :**
> - [Documentation MongoDB officielle](https://www.mongodb.com/docs/v8.0/tutorial/install-mongodb-on-debian/)
> - [Configuration d'un cluster MongoDB](https://www.mongodb.com/resources/products/fundamentals/mongodb-cluster-setup)

### 1.1 Sur le premier nœud MongoDB

#### A. Installer les dépendances

```bash
sudo apt-get install gnupg curl
```

#### B. Importer la clé GPG MongoDB

```bash
curl -fsSL https://www.mongodb.org/static/pgp/server-8.0.asc | \
  sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg --dearmor
```

#### C. Ajouter le dépôt MongoDB

```bash
echo "deb [ signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] http://repo.mongodb.org/apt/debian bookworm/mongodb-org/8.0 main" | \
  sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
```

#### D. Mettre à jour et installer MongoDB

```bash
sudo apt-get update
sudo apt-get install -y mongodb-org
```

#### E. Bloquer les mises à jour automatiques

```bash
sudo apt-mark hold mongodb-org
```

#### F. Configurer MongoDB

Ouvrir le fichier de configuration :

```bash
sudo nano /etc/mongod.conf
```

**Modifier la section `net:` pour écouter sur le réseau :**

Option 1 - Écouter sur toutes les interfaces :
```yaml
net:
  port: 27017
  bindIpAll: true
```

Option 2 - Écouter sur une IP spécifique (exemple : `192.168.140.235`) :
```yaml
net:
  port: 27017
  bindIp: 192.168.140.235
```

Option 3 - Écouter sur un hostname (exemple : `graylog01`) :
```yaml
net:
  port: 27017
  bindIp: graylog01
```

#### G. Démarrer MongoDB

```bash
sudo systemctl daemon-reload
sudo systemctl enable mongod.service
sudo systemctl start mongod.service
```

**Vérification :**
```bash
sudo systemctl status mongod.service
```
---

##  Étape 2 : Installation de Data Node

Data Node gère l'ingestion, le traitement et l'indexation des logs. Il doit être installé sur un **cluster séparé** des nœuds Graylog/MongoDB.

⚠️ **N'INSTALLEZ PAS OpenSearch directement !** Data Node inclut déjà OpenSearch.

### 2.1 Sur le premier nœud Data Node

#### A. Installer le paquet Data Node

```bash
wget https://packages.graylog2.org/repo/packages/graylog-7.0-repository_latest.deb
sudo dpkg -i graylog-7.0-repository_latest.deb
sudo apt-get update
sudo apt-get install graylog-datanode
```

#### B. Configurer vm.max_map_count

OpenSearch nécessite d'augmenter cette limite.

**Vérifier la valeur actuelle :**
```bash
cat /proc/sys/vm/max_map_count
```

**Si inférieur à 262144, augmenter la valeur :**
```bash
echo 'vm.max_map_count=262144' | sudo tee -a /etc/sysctl.d/99-graylog-datanode.conf
sudo sysctl --system
```

**Vérifier que c'est appliqué :**
```bash
cat /proc/sys/vm/max_map_count
```

Devrait afficher : `262144`

#### C. Générer le password_secret

```bash
openssl rand -hex 32
```

**Exemple de sortie :**
```
c0b10ff32ccd565dd76a6331aeba6ff9bce2e6f1c8ca3504092e5856b6c76622
```

**⚠️ CRITIQUE : Sauvegardez ce secret !** Vous devrez l'utiliser dans la configuration Graylog également.

#### D. Configurer Data Node

Ouvrir le fichier de configuration :

```bash
sudo nano /etc/graylog/datanode/datanode.conf
```

**Ajouter/modifier les paramètres suivants :**

1. **Ajouter le password_secret :**
   ```properties
   password_secret = c0b10ff32ccd565dd76a6331aeba6ff9bce2e6f1c8ca3504092e5856b6c76622
   ```

2. **Configurer la mémoire heap (⚠️ non présent par défaut, à ajouter) :**
   
   Règle : 50% de la RAM système, maximum 31 GB.
   
   Exemple pour 8 GB de RAM :
   ```properties
   opensearch_heap = 4g
   ```

3. **Configurer la connexion MongoDB :**
   
   Remplacer `graylog01` par vos hostnames/IPs :
   ```properties
   mongodb_uri = mongodb://graylog01:27017/graylog?replicaSet=rs0
   ```

#### E. Démarrer Data Node

```bash
sudo systemctl daemon-reload
sudo systemctl enable graylog-datanode.service
sudo systemctl start graylog-datanode
```

**Vérification :**
```bash
sudo systemctl status graylog-datanode
```

### 2.2 Répéter sur les autres nœuds Data Node

**⚠️ IMPORTANT :** Répétez les étapes A, B, D et E sur les autres Data Nodes.

**NE PAS générer un nouveau password_secret (étape C) !** Utilisez le **même secret** sur tous les nœuds.

---

## 🎯 Étape 3 : Installation de Graylog Server

Graylog est installé sur les **mêmes nœuds que MongoDB** selon l'architecture Conventionnelle.

### 3.1 Sur le premier nœud Graylog

#### A. Installer le paquet Graylog

**Pour Graylog Open (gratuit) :**
```bash
sudo apt-get install graylog-server
```

**Pour Graylog Enterprise/Security :**
```bash
sudo apt-get install graylog-enterprise
```

#### B. Générer le mot de passe root (administrateur)

```bash
echo -n "MotDePasseAdmin" | sha256sum | cut -d' ' -f1
```

Tapez votre mot de passe (exemple : `Azerty1234!`) et appuyez sur Entrée.

**Exemple de sortie :**
```
86dac5b7dc404d676e3696eb2933734540218c2ea1ed19a4ba9ccb6cd4cad08b
```

**⚠️ IMPORTANT :** Sauvegardez le mot de passe en clair, vous en aurez besoin après la configuration preflight.

#### C. Configurer Graylog

Ouvrir le fichier de configuration :

```bash
sudo nano /etc/graylog/server/server.conf
```

**Modifier les paramètres suivants :**

##### 1. Ajouter le hash du mot de passe root

```properties
root_password_sha2 = 86dac5b7dc404d676e3696eb2933734540218c2ea1ed19a4ba9ccb6cd4cad08b
```

##### 2. Ajouter le password_secret (le même que Data Node !)

```properties
password_secret = c0b10ff32ccd565dd76a6331aeba6ff9bce2e6f1c8ca3504092e5856b6c76622
```

##### 3. Configurer l'adresse d'écoute HTTP

Chercher la section `# HTTP settings` et décommenter/modifier :

```properties
http_bind_address = 0.0.0.0:9000
```

##### 4. Configurer la connexion MongoDB

Remplacer `graylog01` par vos hostname/IPs:

```
mongodb_uri = mongodb://192.168.140.235:27017/graylog
```
##### 5. Configurer le journal (message journal)

Recommandation: 72heure et taille diviser part deux le stockage 

Exemple pour 8gb mettre 4gb

```
message_journal_max_age = 72h
message_journal_max_size = 4gb
```

###### D. Configurer la mémoire heap de Graylog

Ouvrir le fichier de service:
```
sudo nano /etc/default/graylog-server
```
Modifier la ligne GRAYLOG_SERVER_JAVA_OPTS :

Règle : 50% de la RAM système, maximum 16 GB.
Exemple pour 4 GB de RAM (2 GB heap) :
```
GRAYLOG_SERVER_JAVA_OPTS="-Xms2g -Xmx2g -server -XX:+UseG1GC -XX:-OmitStackTraceInFastThrow"
```
Toujours mettre la même valeur pour -Xms et -Xmx !

###### E. Démarrer Graylog

```
sudo systemctl daemon-reload
sudo systemctl enable graylog-server.service
sudo systemctl start graylog-server.service
```
Vérification :

```
sudo systemctl status graylog-server.service
```
Suivre les logs de démarrage :

```
sudo journalctl -u graylog-server -f
```
Attendez le message : Graylog server up and running.

###### Étape 4 : Connexion Preflight

Après l'installation, vous devez effectuer la connexion preflight pour accéder à l'interface Graylog.

###### 4.1 Trouver les identifiants temporaires

Les identifiants de première connexion sont générés automatiquement et affichés dans les logs :

```
sudo nano /var/log/graylog-server/server.log
```
Cherchez une ligne comme :

```
Initial configuration is accessible at 0.0.0.0:9000, with username 'admin' and password 'RdOIRGVrWM'.
```

##### 4.2 Accéder à l'interface

Ouvrez votre navigateur et accédez à votre load balancer ou à l'IP d'un nœud Graylog :

```
http://192.168.140.235:9000
```
##### 4.3 Connexion initiale (preflight)

Première connexion :

Utilisateur : admin
Mot de passe : Le mot de passe temporaire trouvé dans les logs (exemple : RdOIRGVrWM)

##### 4.4 Première interface web 

Création d'un certificat Graylog depuis l'interface web

```
Cliquer sur "Create CA" 
```

Activation et réinitialisation du CA 

```
Laisser les paramètres part défault : Validité de 30jours, renouvelable
```
Importer ensuite le CA pour le Datanode

Configuration importante (à vérifier)

Sur le serveur Graylog, modifier le fichier de configuration suivant :

```
sudo nano /etc/hosts
```

Ajouter l’adresse IP du DataNode et du serveur Graylog, ainsi que leurs noms de machine. 

Exemple
```
127.0.0.1       localhost
192.168.140.235 ORL-SRV-Graylog
192.168.140.10  Datanode
```
Faire la même chose pour le Datatanode, en ajoutant uniquement :

```
127.0.0.1       localhost
192.168.140.10  Datanode
```

Cette configuration est nécessaire afin que le Graylog puisse correctement identifier et communiquer avec le DataNode.

L’interface web de Graylog est accessible via l’URL suivante :

```
http://192.168.140.235:9000
```

Pour en savoir plus sur l'interface web, consulter la documentation suivante : [Graylog-web](./Graylog-web.md) :

