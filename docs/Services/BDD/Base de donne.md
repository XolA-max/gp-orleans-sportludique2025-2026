# Base de Données

---

## Installation de MariaDB

!!! info "Installation"
    ```bash
    sudo apt update && sudo apt upgrade -y
    sudo apt install mariadb-server -y
    ```
    
    Vérification su statut :
    ```bash
    sudo systemctl status mariadb
    ```

---

---

## Configuration

### Création de la base de données et de l’utilisateur

!!! info "SQL"
    Connexion en root :
    ```bash
    sudo mysql -u root -p
    ```
    
    Commandes SQL :
    ```sql
    CREATE DATABASE Wordpress;
    
    CREATE USER 'Nom-utilisateur'@'Adresse_IP_du_serveur_web' IDENTIFIED BY 'Mots_de_passe';
    
    GRANT ALL PRIVILEGES ON Wordpress.* TO 'Nom-utilisateur'@'Adresse_IP_du_serveur_web';
    
    FLUSH PRIVILEGES;
    ```

### Modification de la bind-address

Pour autoriser les connexions distantes.

!!! info
    `sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf`

    Modifier :
    ```ini
    bind-address = 0.0.0.0
    ```
    (Au lieu de `127.0.0.1`)

### Redémarrage du service

!!! info
    ```bash
    sudo systemctl restart mariadb
    ```