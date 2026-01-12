# Interface Web Graylog

---

## 1. Connexion

### Accès à l'interface

Se connecter à l'interface Web (port 9000).

*   **Utilisateur** : `admin`
*   **Mot de passe** : Le hash SHA256 du mot de passe défini lors de l'installation.

!!! info "Retrouver le mot de passe manuellement"
    Si vous avez oublié le mot de passe mais avez accès au serveur, vous pouvez générer le hash d'un nouveau mot de passe :
    ```bash
    echo -n "NouveauMotDePasse" | sha256sum | cut -d' ' -f1
    ```
    *(Ceci ne change pas le mot de passe, c'est juste pour vérifier la valeur à mettre dans server.conf)*

---

---

## 2. Configuration des Inputs

### 2.1 Création d'un Input pour Syslog (Linux)

1.  Aller dans **System** → **Inputs**.
2.  Sélectionner **Syslog UDP** dans la liste dropdown.
3.  Cliquer sur **Launch new input**.

#### Configuration de l'Input

*   **Title** : `Graylog_UDP_Rsyslog_Linux` (exemple)
*   **Port** : `514` (Recommandé car standard)
*   **Bind address** : `0.0.0.0`
*   **Store full message** : Coché (Recommandé)

!!! info "Note"
    Le port 514 est standard pour Syslog. Assurez-vous qu'aucun autre service (comme rsyslog local) n'utilise déjà ce port sur le serveur Graylog, ou utilisez un port différent (ex: 1514).

Cliquer sur **Launch Input**.

### 2.2 Création des Index

L'index est la structure de stockage dans OpenSearch.

1.  Aller dans **System** → **Indices**.
2.  Cliquer sur **Create index set**.
3.  Remplir les champs (Nom, Description, Préfixe).
    *   *Exemple : Index pour Logs Linux.*
4.  Valider.

---

---

## 3. Configuration des Streams

Les Streams permettent de router les logs vers des index spécifiques.

1.  Aller dans **Streams**.
2.  Cliquer sur **Create stream**.
3.  **Nom** : `Linux Stream` (exemple).
4.  **Index Set** : Sélectionner l'index créé précédemment.
5.  Valider.

### Configurer les règles de routage

1.  Cliquer sur **Manage Rules** (ou "More" → "Manage Rules") sur la ligne du Stream.
2.  Cliquer sur **Add stream rule**.
3.  **Field** : `gl2_source_input` (ou autre critère).
4.  **Type** : `match input`.
5.  **Value** : Sélectionner l'Input UDP créé à l'étape 2.1.
6.  Valider.

!!! success "Terminé"
    Start the stream ! N'oubliez pas de cliquer sur **Start Stream** si ce n'est pas automatique.

---

---

## 4. Configuration des Clients Linux (Rsyslog)

Sur la machine cliente (celle qui envoie les logs) :

### A. Installer Rsyslog

!!! info
    ```bash
    sudo apt-get update
    sudo apt-get install rsyslog
    ```

### B. Configurer l'envoi vers Graylog

Créer un fichier de configuration dédié :

!!! info
    `sudo nano /etc/rsyslog.d/10-graylog.conf`

    Ajouter la ligne suivante :
    ```bash
    *.* @192.168.10.220:514;RSYSLOG_SyslogProtocol23Format
    ```
    *(Remplacer 192.168.10.220 par l'IP de votre serveur Graylog)*

### C. Redémarrer Rsyslog

!!! info
    ```bash
    sudo systemctl restart rsyslog
    ```

---

---

## 5. Visualisation des Logs

1.  Aller dans **Streams**.
2.  Cliquer sur le titre de votre Stream (ex: `Linux Stream`).
3.  L'interface de recherche s'ouvre.

### Utilisation de la recherche

*   **Période** : "Last 5 minutes" (par défaut).
*   **Auto-refresh** : Configurable (ex: 5s).
*   **Barre de recherche** :
    *   `source:srv-docker` : Affiche les logs de la machine "srv-docker".
    *   `level:3` : Affiche les erreurs.

!!! tip "Astuce"
    Cliquez sur le bouton "Play" (ou la loupe) pour lancer la recherche après modification des filtres.