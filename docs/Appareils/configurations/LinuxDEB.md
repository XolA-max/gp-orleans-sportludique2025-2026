# Guide linux Debian Prête à l'emplois

--- 

## 1. Commandes Réseau 

### 1.1 Changer l'adresse Ip
!!! info
    ```bash
    sudo nano /etc/network/interfaces
    ```
#### Resultat :

!!! info
    ```bash
    # This file describes the network interfaces available on your system
    # and how to activate them. For more information, see interfaces(5).
    
    source /etc/network/interfaces.d/*
    
    # The loopback network interface
    auto lo
    iface lo inet loopback
    
    # Reseau DMZ
    allow-hotplug ens5
    iface ens5 inet static
            address 192.168.45.2/24
            gateway 192.168.45.254
    
            # dns-* options are implemented by the resolvconf package, if installed
            dns-nameservers 192.168.45.2
            dns-search dns.orleans.sportludique.fr
            up ip route add 172.28.120.0/24 via 192.168.45.1 dev ens5
    
    # Reseau Mana         
    allow-hotplug ens4
    iface ens4 inet static
            address 192.168.140.95/24
    
    ```

>  dns-nameservers : pour definir le DNS 

>  up ip route add : pour ajouter des route statique


### 1.2 Resolution local
!!! info
    ```bash
    /etc/hosts
    
    ```

C’est le fichier de résolution locale de la machine.
Il sert uniquement à associer des noms d’hôtes à des adresses IP sans passer par un serveur DNS.


### 1.3 Changer le nom de la machine 
!!! info
    ```bash
    sudo nano /etc/hostname
    
    ```