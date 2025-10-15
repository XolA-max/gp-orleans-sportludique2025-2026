## Met à jour la liste des paquets disponibles
```
sudo apt update
```
## Met à jour les paquets installés (optionnel, mais recommandé)
```
sudo apt upgrade -y
 ```

 ## Installation d'Apache2
 ```
sudo apt install apache2 -y
```

## Crée le DocumentRoot pour monsite.com
```
sudo mkdir -p /var/www/siteorl/html
```
### Exemple de configuration

#### Fichier : /etc/apache2/sites-available/siteorl.orleans.sportludique.fr.conf 
#### ou 
#### etc/apache2/sites-available/[monsite.conf]
```
<VirtualHost *:80>
        
        ServerName www.orleans.sportludique.fr 
        ServerAlias www.orleans.sportludique.fr
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
## Activer un vhost 
(lier /etc/apache2/sites-available/siteorl.orleans.sportludique.fr.conf et /etc/apache2/sites-enable/siteorl.orleans.sportludique.fr.conf)
```
sudo a2ensite siteorl.orleans.sportludique.fr.conf 
ou
sudo a2ensite [monsite.conf]
```
