# DNS Secondaire

Guide de configuration DNS autoritaire secondaire et DNS redirecteur secondaire.

---

---

## DNS Esclave (Autorité Secondaire)

### Configuration des Zones par Défaut

!!! warning
    Sur le serveur secondaire, les zones par défaut doivent être commentées si elles ne sont pas utilisées ou en conflit avec les vues.

    `sudo nano /etc/bind/named.conf.default-zones`

    ```bind
    // prime the server with knowledge of the root servers
    //zone "." { ... };
    
    // be authoritative for the localhost forward and reverse zones
    //zone "localhost" { ... };
    
    // ... (tout commenter)
    ```

### Configuration des Vues (Views)

Les vues doivent être configurées pour agir comme esclaves (`type slave`) et pointer vers le maître (`masters`).

!!! info
    `sudo nano /etc/bind/named.conf.local`

    ```bind
    // Zone externe
    view "outside" {
        match-clients {
            !172.x.x.0/24;      // Pas du LAN
            !192.168.x.0/24;    // Pas de la DMZ
            127.0.0.1;
            any;                // Tout le reste
        };
        zone "orleans.sportludique.fr." {
            type slave;
            file "/etc/cache/bind/db.orleans.sp.fr.externe";
            masters { 192.168.45.2; };
        };
    };

    // Zone interne
    view "inside" {
        match-clients {
            172.x.0.0/24;       // LAN
            192.168.x.0/24;     // DMZ
            127.0.0.1;
        };
        zone "orleans.sportludique.fr." {
            type slave;
            file "/etc/cache/bind/db.orleans.sp.fr.interne";
            masters { 192.168.45.2; };
        };
    };
    ```

### Fichiers de Zone

!!! info "Note Importante"
    Le DNS Autorité secondaire n'a pas besoin d'avoir de configuration manuelle dans les fichiers `/etc/bind/db.orleans.sp.fr.interne` et `/etc/bind/db.orleans.sp.fr.externe`.
    
    L'autorité principale va transférer les zones au secondaire (transfert de zone), et Bind remplira ces fichiers automatiquement (format souvent binaire/illisible).

### Configuration des Options

!!! info
    `sudo nano /etc/bind/named.conf.options`

    ```bind
    options {
            directory "/var/cache/bind";
    }
    ```

### Résolveur

!!! info
    `sudo nano /etc/resolv.conf`
    
    ```bind
    search orleans.sportludique.fr
    nameserver 172.28.120.3
    ```

### Configuration Locale

!!! warning
    Ne pas oublier de remplir `/etc/bind/db.local` si nécessaire pour que le serveur Apache retourne des SERVFAIL correctement en cas de problème.

---

---

## DNS Résolveur Secondaire

### Configuration des Options

!!! info
    `sudo nano /etc/bind/named.conf.options`

    ```bind
    options {
            directory "/var/cache/bind";
            
            // Autorise les requetes recursives
            recursion yes;  
            
            // Liste des réseaux autorisés à interoger le resolver
            allow-query {
                172.16.x.0/24;      // LAN
                127.0.0.1;          // LOCALHOST
                192.168.x.0/24;     // DMZ
            };
            
            // Desactivation de DNSSec
            dnssec-validation no;
            
            // Ecoute sur l'ensemble des interfaces IPv4
            listen-on { any; };
    };
    ```

### Configuration Locale (Zones Forward)

Les requêtes locales sont redirigées vers le DNS de la DMZ (Vue Interne).

!!! info
    `sudo nano /etc/bind/named.conf.local`

    ```bind
    zone "orleans.sportludique.fr" {
        type forward;
        forwarders { 192.168.x.y };  // IP du serveur DNS Auto DMZ
    };
    ```

Les requêtes vers `sportludique.fr` sont redirigées vers le serveur de l'enseignant.

!!! info
    ```bind
    zone "sportludique.fr" {
        type forward;
        forwarders { 121.183.90.205; };
    };
    ```

### Configuration Résolveur Système

!!! info
    `sudo nano /etc/resolv.conf`

    ```bind
    search orleans.sportludique.fr
    nameserver 192.168.45.2
    nameserver 192.168.45.4
    ```