# Guide de configuration Reverse Proxy secondaire
## Reverse Proxy Secondaire
 
# Synchronisation de configuration Nginx entre Reverse Proxy
 
## Table des matières
 
## 1. Prérequis
 
- Deux serveurs Reverse Proxy configurés
- Accès root ou sudo sur les deux serveurs
- Nginx installé sur les deux serveurs ([voir installation](#1-installation-de-nginx))
 
## 2. Installation de rsync et lsyncd
 
Il faut installer ces paquets sur **les deux Reverse Proxy**.
 
```bash
sudo apt install rsync lsyncd
```
 
## 3. Configuration de lsyncd
 
### 3.1 Création de la configuration
 
Créer ou éditer le fichier de configuration `/etc/lsyncd/lsyncd.conf.lua` :
 
```lua
  GNU nano 7.2                                  /etc/lsyncd/lsyncd.conf.lua                                           
settings {
  logfile    = "/var/log/lsyncd.log",
  statusFile = "/var/log/lsyncd.status",
  statusInterval = 20,
  inotifyMode = "CloseWrite"
}
 
sync {
  default.rsync,
  source = "/etc/nginx/",
  target = "etudiant@192.168.45.8:/etc/nginx/",
  rsync = {
    archive = true,
    compress = false,
    verbose = true,
    rsh = "/usr/bin/ssh -o StrictHostKeyChecking=no",
    _extra = "--rsync-path='sudo rsync'",
    _extra = "--delete"   
  },
  delay = 2
}
 
```
 
> **Note :** Remplacez `192.168.45.8` par l'adresse IP de votre Reverse Proxy secondaire.
 
### 4.2 Démarrage du service
 
Redémarrer et activer le service lsyncd :
 
```bash
sudo systemctl restart lsyncd
sudo systemctl enable lsyncd
```
 
## 5. Test de synchronisation
 
### 5.1 Création d'un fichier test
 
Sur le **Reverse Proxy Primaire** :
 
```bash
sudo touch /etc/nginx/test-sync.txt
```
 
### 5.2 Vérification sur le secondaire
 
Puis sur le **Reverse Proxy Secondaire** :
 
```bash
ls -l /etc/nginx/
```
 
Si le fichier `test-sync.txt` apparaît, la synchronisation est fonctionnelle.
 
## 6. Configuration HTTPS
 
### 6.1 Important concernant les certificats SSL
 
**⚠️ Attention :** La synchronisation lsyncd ne gère **pas** automatiquement les certificats SSL. Il faut les importer manuellement sur le Reverse Proxy secondaire.
 
### 6.2 Configuration pour HTTPS
 
Après avoir obtenu vos certificats SSL (via Let's Encrypt ou autre), modifier la configuration du site :
 
```bash
sudo nano /etc/nginx/sites-available/orleans.sportludique.fr
```
 
Exemple de configuration HTTPS :
 
```nginx
server {
    listen 80;
    server_name orleans.sportludique.fr;
    return 301 https://$server_name$request_uri;
}
 
server {
    listen 443 ssl;
    server_name orleans.sportludique.fr;
    
    ssl_certificate /etc/letsencrypt/live/orleans.sportludique.fr/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/orleans.sportludique.fr/privkey.pem;
    
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384;
    
    location / {
        proxy_pass http://backend_server;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
 
### 6.3 Copie des certificats sur le secondaire
 
Les certificats doivent être copiés manuellement sur le Reverse Proxy secondaire :
 
```bash
# Sur le Reverse Proxy Primaire
sudo scp -r /etc/letsencrypt/live/orleans.sportludique.fr root@172.28.62.10:/etc/letsencrypt/live/
sudo scp -r /etc/letsencrypt/archive/orleans.sportludique.fr root@172.28.62.10:/etc/letsencrypt/archive/
```
 
### 6.4 Activation de la configuration HTTPS
 
Après modification, activer la configuration si ce n'est pas déjà fait :
 
```bash
sudo ln -s /etc/nginx/sites-available/orleans.sportludique.fr /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```
 
## 7. Récapitulatif des étapes
 
1. Installer Nginx, rsync et lsyncd sur les deux serveurs
2. Configurer lsyncd sur le serveur primaire
3. Créer la configuration du site dans `/etc/nginx/sites-available/`
4. Créer le lien symbolique vers `/etc/nginx/sites-enabled/`
5. Pour HTTPS : copier manuellement les certificats SSL sur le secondaire
6. Tester et recharger Nginx
 
## 8. Ressources complémentaires
 
Pour plus d'informations sur la gestion des certificats SSL, consultez la documentation appropriée de votre autorité de certification (Let's Encrypt, etc.).
 
 
 