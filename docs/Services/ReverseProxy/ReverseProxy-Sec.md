# Reverse Proxy Secondaire & Synchronisation

---

## 1. Prérequis

*   Deux serveurs Reverse Proxy configurés (Nginx installé).
*   Accès root ou sudo sur les deux serveurs.

---

---

## 2. Installation des outils

Sur **les deux serveurs** :

!!! info
    ```bash
    sudo apt install rsync lsyncd
    ```

---

---

## 3. Configuration de lsyncd (Serveur Primaire)

`lsyncd` va surveiller les modifications de configuration sur le primaire et les répliquer sur le secondaire.

Créer le fichier de configuration :

!!! info
    `sudo nano /etc/lsyncd/lsyncd.conf.lua`

    ```lua
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
    *(Remplacer 192.168.45.8 par l'IP du secondaire)*

### Démarrage

!!! info
    ```bash
    sudo systemctl restart lsyncd
    sudo systemctl enable lsyncd
    ```

---

---

## 4. Test de synchronisation

Sur le **Primaire** :
```bash
sudo touch /etc/nginx/test-sync.txt
```

Sur le **Secondaire** :
```bash
ls -l /etc/nginx/
```
*(Le fichier doit apparaître)*

---

---

## 5. Gestion HTTPS et Certificats

!!! warning "Important"
    La synchronisation `lsyncd` ne gère **PAS** automatiquement les certificats SSL situés dans `/etc/letsencrypt`. Il faut les copier manuellement.

### Synchronisation manuelle des certificats

Depuis le Primaire vers le Secondaire :

!!! info
    ```bash
    sudo scp -r /etc/letsencrypt/live/orleans.sportludique.fr root@172.28.62.10:/etc/letsencrypt/live/
    sudo scp -r /etc/letsencrypt/archive/orleans.sportludique.fr root@172.28.62.10:/etc/letsencrypt/archive/
    ```

### Configuration Nginx HTTPS

Exemple de bloc server :

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
    
    location / {
        proxy_pass http://backend_server;
        proxy_set_header Host $host;
        # ... autres headers
    }
}
```

Recharger Nginx après copie des certificats :

!!! info
    ```bash
    sudo nginx -t
    sudo systemctl reload nginx
    ```