# Guide mise en place d'un Reverse Proxy avec Nginx

## Introduction

Un **reverse proxy** est un serveur intermédiaire qui reçoit les
requêtes des clients et les redirige vers un ou plusieurs serveurs
internes.\
Il permet : - de sécuriser les services internes, - de centraliser
l'accès, - d'ajouter du HTTPS, - de répartir la charge entre plusieurs
serveurs backend, - d'améliorer les performances via la mise en cache.

Cette documentation décrit toutes les étapes nécessaires pour installer
et configurer un reverse proxy Nginx avec gestion de TLS/HTTPS.

------------------------------------------------------------------------

## 1. Installation de Nginx

### Sur Debian/Ubuntu :

``` bash
sudo apt update
sudo apt install nginx
```

------------------------------------------------------------------------

## 2. Structure de base d'un Reverse Proxy

Nginx fonctionne avec des blocs **server** dans
`/etc/nginx/sites-available`.

Exemple minimal :

``` nginx
server {
    listen 80;
    server_name monsite.domaine.com;

    location / {
        proxy_pass http://192.168.1.20;
        include /etc/nginx/proxy_params;
    }
}
```

------------------------------------------------------------------------

## 3. Activation de la configuration

Créez le fichier :

``` bash
sudo nano /etc/nginx/sites-available/reverseproxy.conf
```

Activez-le :

``` bash
sudo ln -s /etc/nginx/sites-available/reverseproxy.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

------------------------------------------------------------------------

## 4. Configuration des paramètres proxy

Nginx utilise un fichier `proxy_params` :

``` nginx
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
```

------------------------------------------------------------------------

## 5. Mise en place de HTTPS (TLS)

### Étapes :

1.  Obtenir un certificat SSL.
2.  Mélanger certificat + clé privée dans un fichier PEM si nécessaire.
3.  Ajouter un bloc `server` HTTPS dans Nginx.
4.  Forcer la redirection HTTP → HTTPS.

------------------------------------------------------------------------

## 6. Combiner certificat + clé privée dans un fichier PEM

Dans certains cas, notamment pour certains outils ou pour faciliter
l'intégration, il est possible (bien que non obligatoire) de **combiner
la clé privée et le certificat dans un même fichier PEM**.

⚠️ **Attention : ce fichier contient à la fois le certificat et la clé
privée --- il doit être protégé !**

### Exemple de création d'un fichier combiné :

``` bash
cat certificat.crt cle_privee.key > certificat_combine.pem
```

Le fichier final ressemblera à ceci :

    -----BEGIN CERTIFICATE-----
    ... contenu du certificat ...
    -----END CERTIFICATE-----
    -----BEGIN PRIVATE KEY-----
    ... contenu de la clé privée ...
    -----END PRIVATE KEY-----

Ensuite, dans votre configuration Nginx, vous pouvez l'utiliser :

``` nginx
ssl_certificate     /etc/ssl/certs/certificat_combine.pem;
ssl_certificate_key /etc/ssl/certs/certificat_combine.pem;
```

💡 *Nginx accepte le même fichier pour les deux directives tant que la
clé et le certificat sont présents dans le même fichier.*

------------------------------------------------------------------------

## 7. Exemple complet d'une configuration HTTPS

``` nginx
server {
    listen 80;
    server_name monsite.domaine.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name monsite.domaine.com;

    ssl_certificate /etc/ssl/certs/certificat_combine.pem;
    ssl_certificate_key /etc/ssl/certs/certificat_combine.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;

    location / {
        proxy_pass http://192.168.1.20;
        include /etc/nginx/proxy_params;
    }
}
```

------------------------------------------------------------------------

## 8. Dépannage

### Voir les logs :

``` bash
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log
```

### Tester le proxy :

``` bash
curl -I https://monsite.domaine.com
```

------------------------------------------------------------------------

## Résumé

-   Installation et configuration du reverse proxy.
-   Activation des paramètres proxy.
-   Mise en place de HTTPS.
-   **Possibilité de combiner certificat + clé privée dans un fichier
    PEM unique**.
-   Vérification et tests.

------------------------------------------------------------------------

## Fin de la documentation