# Guide Reverse Proxy Nginx

---

## 1. Installation

### Sur Debian/Ubuntu

!!! info
    ```bash
    sudo apt update
    sudo apt install nginx
    ```

---

---

## 2. Configuration de base

Les configurations se trouvent dans `/etc/nginx/sites-available/`.

### Exemple HTTP simple

!!! info "reverseproxy.conf"
    ```nginx
    server {
        listen 80;
        server_name monsite.domaine.com;

        location / {
            proxy_pass http://192.168.1.20;
            include /etc/nginx/proxy_params;
        }
    }
    ```

### Activation

!!! info
    ```bash
    sudo ln -s /etc/nginx/sites-available/reverseproxy.conf /etc/nginx/sites-enabled/
    sudo nginx -t
    sudo systemctl reload nginx
    ```

---

---

## 3. Paramètres Proxy

Créer ou vérifier `/etc/nginx/proxy_params` :

```nginx
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
```

---

---

## 4. Configuration HTTPS (TLS)

### 4.1 Certificats combinés (Optionnel)

Vous pouvez combiner certificat et clé privée dans un seul fichier PEM (bien que non standard, Nginx le supporte tant que la clé est lisible).

!!! info
    ```bash
    cat certificat.crt cle_privee.key > certificat_combine.pem
    ```

    *Attention à protéger ce fichier !*

### 4.2 Configuration Nginx HTTPS

!!! info "Configuration Complète"
    ```nginx
    server {
        listen 80;
        server_name monsite.domaine.com;
        return 301 https://$host$request_uri;
    }

    server {
        listen 443 ssl;
        server_name monsite.domaine.com;

        # Certificats
        ssl_certificate /etc/ssl/certs/certificat_combine.pem;
        ssl_certificate_key /etc/ssl/certs/certificat_combine.pem;

        # Sécurité SSL
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_prefer_server_ciphers on;

        location / {
            proxy_pass http://192.168.1.20;
            include /etc/nginx/proxy_params;
        }
    }
    ```

---

---

## 5. Dépannage

### Logs

!!! info
    ```bash
    tail -f /var/log/nginx/access.log
    tail -f /var/log/nginx/error.log
    ```

### Test

!!! info
    ```bash
    curl -I https://monsite.domaine.com
    ```