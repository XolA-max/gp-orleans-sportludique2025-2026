# Guide de configuration DNS autoritaire et DNS
## DNS-Autorité-Secondaire
 
>
>
 
## DNS étant esclaves du DNS Autorite
## Fichier /etc/bind/named.conf.default-zones
```markdown
// prime the server with knowledge of the root servers
//zone "." {
//      type hint;
//      file "/usr/share/dns/root.hints";
//};
 
// be authoritative for the localhost forward and reverse zones, and for
// broadcast zones as per RFC 1912
 
//zone "localhost" {
//      type slaves;
//      file "/etc/bind/db.local";
//};
 
//zone "127.in-addr.arpa" {
//      type slaves;
//      file "/etc/bind/db.127";
//};
 
//zone "0.in-addr.arpa" {
//      type slaves;
//      file "/etc/bind/db.0";
//};
 
//zone "255.in-addr.arpa" {
//      type slaves;
//      file "/etc/bind/db.255";
//};
 
```
 
## Fichier /etc/bind/named.conf.local
 
```markdown
    //zone externe
    view "outside" {
        match-clients {
            !172.x.x.0/24;    //requète de provenant pas du LAN
            !192.168.x.0/24;    //requète de provenant pas de la DMZ
            127.0.0.1;
            any;                //Toutes les autres adresses
        };
        zone "orleans.sportludique.fr." {
            type slave;
            file "/etc/cache/bind/db.orleans.sp.fr.externe";
            masters { 192.168.45.2; };
        };
    };
    view "inside" {
        match-clients {
            172.x.0.0/24;    //requète provenant du LAN
            192.168.x.0/24;    //requète provenant de la DMZ
            127.0.0.1;
        };
        zone "orleans.sportludique.fr." {
            type slave;
            file "/etc/cache/bind/db.orleans.sp.fr.interne";
            masters { 192.168.45.2; };
        };
    };
 
```
 
## Fichier /etc/bind/db.orleans.sp.fr.interne et /etc/bind/db.orleans.sp.fr.externe
<aside>
 
Le DNS Autorité secondaire n'a pas besoin d'avoir de configuration dans les fichiers (/etc/bind/db.orleans.sp.fr.interne et /etc/bind/db.orleans.sp.fr.externe) l'autorité principale va répondre au secondaire et ainsi il remplira les fichiers automitequement, ils seront illisible.
 
<aside>
 
 
## Fichier /etc/bind/named.conf.options
 
```markdown
    options {
            directory "/var/cache/bind";
    }
 
```
 
## Fichier /etc/resolv.conf
 
```markdown
    search orleans.sportludique.fr
    nameserver 172.28.120.3
 
```

## DNS Résolveur Secondaire
 
### Fichier /etc/bind/named.conf.options
 
```markdown
    options {
            directory "/var/cache/bind";
            //Autorise les requetes recursives
            recursion yes;  
            //Liste des réseaux autorisés à interoger le resolver
            //Par defaut seul les équipements du meme réseau IP que le serveur peuvent l\'interroger.
            allow-query {
                172.16.x.0/24;       //LAN
                127.0.0.1;          //LOCALHOST
                192.168.x.0/24;    //DMZ
            };
            //Desactivation de DNSSec
            dnssec-validation no;
            //Ecoute sur l\'ensemble des interfaces IPv4
            listen-on { any; };
    };
```
### Fichier /etc/bind/named.conf.local (du resolver)
 
Les requetes à destination de la zone locale doivent être redirigées
vers le serveur DNS de la DMZ en interogeant la vue interne.
 
```markdown
zone "orleans.sportludique.fr" {
    type forward;
    forwarders { 192.168.x.y };  // Remplace par l'IP du serveur DNS ayant autorité sur la zone (dans la DMZ par exemple)
};
```
 
Les requetes à destination de la zone de l'entreprise
(sportludique.fr) doivent partir vers le serveur resolver de
l'enseignant gérant cette zone.
 
```markdown
zone "sportludique.fr" {
    type forward;
    forwarders { 121.183.90.205; };  // IP du serveur de l'enseignant ayant autorité sur la zone sportludique.fr
};
```
 
## Fichier /etc/resolv.conf
 
```markdown
search orleans.sportludique.fr
nameserver 192.168.45.2
nameserver 192.168.45.4
 
```