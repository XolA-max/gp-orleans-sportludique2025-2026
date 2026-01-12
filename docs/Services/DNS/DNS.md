# Guide Installation DNS

> Linux → bind9

## Installation

!!! info
    Installation des paquets :
    ```bash
    sudo apt install -y bind9 bind9utils bind9-dnsutils
    ```

---

---

## Configuration BIND (Zone Principale)

### Exemple pour `orleans.sportludique.fr`

Dans le fichier de zone, ajoutez les enregistrements NS et A nécessaires.

!!! info
    Fichier de zone (exemple) :
    
    ```bind
    ; Fichier de zone pour exemple.com (zone publique)
    $TTL 3600
    @      IN      SOA    dns.orleans.sportludique.fr. ns1.exemple.com. admin.exemple.com. (
                      2023091301 ; Numéro de série
                      3600       ; Intervalle de rafraîchissement
                      1800       ; Intervalle de réessai
                      604800     ; Intervalle d'expiration
                      86400 )    ; Durée minimale de cache
    @      IN      NS     dns.orleans.sportludique.fr. 
    @      IN      NS     ns2.exemple.com.
    
    ; Enregistrements IPv4
    @      IN      A      [Adresse ip d'un serveur]
    www    IN      A      [Adresse ip du serv web]
    mail   IN      A      [Adresse ip du serv mail]
    dns    IN      A      192.168.45.2
    
    ; Délégation du sous-domaine
    sousdomaine.exemple.com.  IN  NS  ns.serveurdns.externe.com.
    ```

---

---

## Résolveur DNS (VM-1)

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

Les requêtes à destination de la zone locale doivent être redirigées vers le serveur DNS de la DMZ (vue interne).

!!! info
    `sudo nano /etc/bind/named.conf.local`

    ```bind
    zone "orleans.sportludique.fr" {
        type forward;
        forwarders { 192.168.x.y };  // IP du serveur DNS autorité DMZ
    };
    ```

Les requêtes vers `sportludique.fr` (entreprise) sont redirigées vers le serveur de l'enseignant.

!!! info
    ```bind
    zone "sportludique.fr" {
        type forward;
        forwarders { 121.183.90.205; };  // IP du serveur de l'enseignant
    };
    ```

---

---

## DNS Autorité (VM-2)

### Fichier `/etc/bind/named.conf.default-zones`

!!! warning
    Toutes les lignes du fichier `named.conf.default-zones` doivent être commentées car elles ne sont associées à aucune vue spécifique ici.

    ```bind
    // zone "." { ... };
    // zone "localhost" { ... };
    // ...
    ```

### Configuration des Vues (Views)

!!! info "Définition : Vue (View)"
    Une vue permet à un même serveur DNS de répondre différemment selon l’origine de la requête (LAN, DMZ, Externe).

!!! info
    `sudo nano /etc/bind/named.conf.local`

    ```bind
    // Zone externe
    view "outside" {
        match-clients {
            !172.x.x.0/24;      // Pas du LAN
            !192.168.x.0/24;    // Pas de la DMZ
            any;                // Tout le reste
        };
        zone "orleans.sportludique.fr." {
            type master;
            file "/etc/bind/db.orleans.sp.fr.externe";
        };
    };

    // Zone interne
    view "inside" {
        match-clients {
            172.x.0.0/24;       // LAN
            192.168.x.0/24;     // DMZ
        };
        zone "orleans.sportludique.fr." {
            type master;
            file "/etc/bind/db.orleans.sp.fr.interne";
        };
    };
    ```

### Fichiers de Zone

#### Vue Interne

!!! info
    `sudo nano /etc/bind/db.orleans.sp.fr.interne`

    ```bind
    ; BIND data file for internal view
    $TTL    604800
    @       IN      SOA     ns1.orleans.sportludique.fr. root.orleans.sportludique.fr. (
                             2025101301   ; Serial
                             604800       ; Refresh
                              86400       ; Retry
                            2419200       ; Expire
                             604800 )     ; Negative Cache TTL
    ;
    @       IN      NS      ns1.orleans.sportludique.fr.
    ns1     IN      A       192.168.45.2
    www     IN      A       192.168.45.3
    ```

#### Vue Externe

!!! info
    `sudo nano /etc/bind/db.orleans.sp.fr.externe`

    ```bind
    ; BIND data file for external view
    $TTL    604800
    @       IN      SOA   ns1.orleans.sportludique.fr. root.orleans.sportludique.fr. (
                         2025150938     ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
    ;
    @       IN      NS      ns1.orleans.sportludique.fr.

    @       IN      A       183.44.45.1
    ns1     IN      A       183.44.45.1
    www     IN      A       183.44.45.1
    ```

### Explications des enregistrements

!!! info
    * **SOA (Start Of Authority)** : Définit le serveur DNS maître et les paramètres de la zone.
    * **NS (Name Server)** : Désigne les serveurs DNS autoritaires.
    * **A (Address)** : Associe un nom d’hôte à une IPv4.
    * **CNAME** : Crée un alias vers un autre nom canonique.

---

---

## Route Interne

Configuration pour le routage interne.

!!! info
    `sudo nano /etc/network/interfaces`

    ```bash
    up ip route add 172.28.96.0/24 via 192.168.45.1 dev ens5
    ```

---

## Logs

Ajouter des logs personnalisés pour le DNS.

!!! info
    `sudo nano /etc/bind/named.conf.options`

    ```bind
    logging {
        channel queries_log {
                    file "/var/log/named/queries.log" versions 3 size 10m;
                    severity info;
                    print-time yes;
             };
             category queries { queries_log; };
             category query-errors { queries_log; };
    };
    ```

    Pour lire les logs :
    ```bash
    sudo tail -f /var/log/named/queries.log
    ```

---

## Vérification

Commandes utiles pour vérifier la configuration.

!!! info
    ```bash
    sudo named-checkconf
    nslookup ns1.orleans.sportludique.fr  
    dig dns.orleans.sportludique.fr
    ```
