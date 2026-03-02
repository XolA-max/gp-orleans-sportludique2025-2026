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

## 2. Sécurisation de l'accès SSH

Afin de sécuriser les accès aux machines, l'authentification par mot de passe est désactivée au profit d'une authentification par clé publique (protocole SSH).

### 2.1 Générer une paire de clés (sur votre PC)
Si vous ne possédez pas encore de clé SSH, ouvrez un terminal (PowerShell sous Windows ou terminal Linux/macOS) et tapez :
!!! info
    ```bash
    ssh-keygen -t ed25519 -C "votre_email@exemple.com"
    ```
Laissez l'emplacement par défaut et définissez une *passphrase* si vous le souhaitez.

### 2.2 Copier la clé publique sur la VM
Pour autoriser votre PC à se connecter à la VM, vous devez copier votre clé **publique** (`.pub`) sur celle-ci.

**Sous Linux/macOS :**
!!! info
    ```bash
    ssh-copy-id utilisateur@ip_de_la_vm
    ```

**Sous Windows (PowerShell) :**
!!! info
    ```powershell
    type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh utilisateur@ip_de_la_vm "cat >> .ssh/authorized_keys"
    ```
*(Note : il peut être nécessaire de créer le dossier `.ssh` sur la VM au préalable avec `mkdir -p ~/.ssh`)*

### 2.3 Désactiver l'authentification par mot de passe
Sur la VM Debian, modifiez la configuration du service SSH :
!!! info
    ```bash
    sudo nano /etc/ssh/sshd_config
    ```

Repérez et modifiez les lignes suivantes (décommentez-les si nécessaire) :
!!! info
    ```ini
    # Désactiver l'authentification par mot de passe
    PasswordAuthentication no
    
    # S'assurer que l'authentification par clé est activée
    PubkeyAuthentication yes
    
    # (Optionnel) Bloquer l'accès direct en root
    PermitRootLogin prohibit-password
    ```

### 2.4 Appliquer les changements
Redémarrez le service SSH pour prendre en compte la nouvelle configuration :
!!! info
    ```bash
    sudo systemctl restart ssh
    ```
> **⚠️ Attention :** Avant de fermer votre session actuelle, ouvrez un second terminal et vérifiez que vous parvenez bien à vous connecter via votre clé SSH. Si ce n'est pas le cas, vous risquez de perdre l'accès en modifiant la configuration !