# Guide Apache2

## Mise à jour du système

!!! info
    Met à jour la liste des paquets disponibles :
    ```bash
    sudo apt update
    ```

    Met à jour les paquets installés (optionnel, mais recommandé) :
    ```bash
    sudo apt upgrade -y
    ```

---

## Installation d'Apache2

!!! info
    ```bash
    sudo apt install apache2 -y
    ```

---

---

## Configuration des VHosts

Sous Linux la configuration des Vhosts se fait dans l'arborescence `/etc/apache2/`.

!!! info
    * `/etc/apache2/sites-available/` : contient tous les fichiers de configuration des sites.
    * `/etc/apache2/sites-enabled/` : contient les sites actuellement activés (liens symboliques pointant vers `sites-available`).

    **Fichier d'exemple :** `/etc/apache2/sites-available/www.orleans.sportludique.fr.conf`

### Configuration du VHost

!!! info
    `sudo nano /etc/apache2/sites-available/www.orleans.sportludique.fr.conf`

    ```apache
    <VirtualHost *:80>
            # The ServerName directive sets the request scheme, hostname and port that
            # the server uses to identify itself.
            ServerName www.orleans.sportludique.fr
            ServerAlias orleans.sportludique.fr
            DocumentRoot /var/www/siteorl

            <Directory /var/www/siteorl>
                Options Indexes FollowSymLinks
                AllowOverride All
                # Require all granted
            </Directory>

            ErrorLog ${APACHE_LOG_DIR}/error.log
            CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
    ```

### Activer un VHost

Une fois le fichier de configuration créé, il faut l'activer et recharger Apache.

!!! info
    ```bash
    sudo a2ensite www.orleans.sportludique.fr.conf
    sudo systemctl reload apache2
    ```

---

---

## Création du site web

Création des répertoires et de la page d'accueil pour `www.orleans.sportludique.fr`.

!!! info
    Création de l'arborescence :
    ```bash
    sudo mkdir -p /var/www/siteorl
    ```

    Création de la page d'index :
    ```bash
    sudo nano /var/www/siteorl/index.html
    ```

    Contenu HTML :
    ```html
    <!DOCTYPE html>
    <html lang="fr">
    <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Accueil - SportLudique</title>
      <style>
        /* Style de base */
        body {
          margin: 0;
          font-family: Arial, sans-serif;
          background-color: #000; /* fond sombre */
          color: white;
          text-align: center;
        }

        /* Bandeau supérieur */
        header {
          background: linear-gradient(90deg, #0B7A2B, #10A339, #00C27A);
          padding: 15px;
          font-size: 1.2em;
          font-weight: bold;
        }

        /* Contenu principal */
        main {
          padding: 60px 20px;
        }

        a {
          display: inline-block;
          background-color: #10A339;
          color: white;
          text-decoration: none;
          padding: 10px 20px;
          border-radius: 6px;
          margin-top: 20px;
          transition: background 0.3s;
        }

        a:hover {
          background-color: #0B7A2B;
        }
      </style>
    </head>
    <body>

      <header>🏋️‍♂️ SportLudique Project 2025-2026</header>

      <main>
        <h1>Bienvenue sur le projet SportLudique</h1>
        <p>Projet collaboratif des étudiants du BTS SIO – spécialité SISR</p>
        <a href="https://lmeryfulbert.github.io/SportLudique2025-2026/" target="_blank">
          Voir la Documentation
        </a>
      </main>

    </body>
    </html>
    ```

---

---

## Configuration Route Statique (Interne)

Pour permettre l'accès au site depuis le réseau LAN, on ajoute une route statique.

!!! info
    `sudo nano /etc/network/interfaces`

    ```bash
    auto lo
    iface lo inet loopback
    
    allow-hotplug ens3
    iface ens3 inet static
            address 192.168.45.3/24
            gateway 192.168.45.254
            dns-nameservers 172.28.120.2
            dns-search orl.orleans.sportludique.fr
            up ip route add 172.28.96.0/19 via 192.168.45.1 dev ens3
    
    allow-hotplug ens4
    iface ens4 inet static
            address 192.168.140.105/24
    ```

    Explication :
    ```text
    up ip route add 172.28.96.0/19 via 192.168.45.1 dev ens3
    ```
    Cela indique : pour atteindre le réseau `172.28.96.0/19`, passer par la passerelle `192.168.45.1`, en utilisant l’interface `ens3`.

---

## Configuration Complémentaire

### Sur les routeurs
!!! warning
    Penser à rediriger le port **80** vers le serveur web (NAT/PAT).

### Sur le serveur DNS
!!! warning
    Ajouter l'enregistrement DNS :
    ```dns
    www IN A 192.168.45.3
    ```
