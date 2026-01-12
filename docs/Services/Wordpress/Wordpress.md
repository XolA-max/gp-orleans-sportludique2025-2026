# Guide d'installation WordPress

---

## 1. Installation des Prérequis

Installation de MariaDB et des extensions PHP nécessaires.

!!! info
    ```bash
    sudo apt update
    sudo apt install mariadb-server -y
    sudo apt install php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip -y
    ```

---

---

## 2. Installation de WordPress

Téléchargement et mise en place des fichiers.

!!! info
    ```bash
    cd /tmp
    wget https://wordpress.org/latest.zip
    
    # Nettoyage index par défaut
    sudo rm /var/www/html/index.html
    
    # Installation utilitaire zip
    sudo apt install zip -y
    
    # Décompression
    sudo unzip latest.zip -d /var/www/html
    
    # Déplacement à la racine web
    sudo mv /var/www/html/wordpress/* /var/www/html/
    sudo rmdir /var/www/html/wordpress
    
    # Permissions
    sudo chown -R www-data:www-data /var/www/html/
    ```

---

---

## 3. Configuration de WordPress

Création du fichier de configuration.

!!! info
    ```bash
    sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
    sudo nano /var/www/html/wp-config.php
    ```

### Paramètres Base de Données

Modifier les lignes suivantes avec vos informations :

```php
// ** Réglages MySQL ** //
/** Nom de la base de données de WordPress */
define( 'DB_NAME', 'nom_de_la_base' );

/** Utilisateur MySQL */
define( 'DB_USER', 'nom_utilisateur' );

/** Mot de passe MySQL */
define( 'DB_PASSWORD', 'mot_de_passe' );

/** Adresse de l’hôte MySQL */
define( 'DB_HOST', 'localhost' ); // ou IP du serveur BDD

/** Jeu de caractères */
define( 'DB_CHARSET', 'utf8' );

/** Méthode d’accès au système de fichiers */
define( 'FS_METHOD', 'direct' );
```

---

---

## 4. Finalisation

Redémarrer les services pour appliquer tous les changements.

!!! info
    ```bash
    sudo systemctl restart apache2
    sudo systemctl restart mariadb
    ```