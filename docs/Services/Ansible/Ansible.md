# Guide d'installation Ansible

1. Installation

Depuis une machine Linux / Debian, mettre à jour le système :

sudo apt update
sudo apt upgrade -y

Installer Ansible.

    sudo apt install ansible -y

2. Configuration de base pour test

Créer le fichier d’inventaire :

    sudo mkdir -p /etc/ansible
    
    sudo nano /etc/ansible/hosts

Exemple simple de contenu pour tester sur la machine locale :

    [local]
    127.0.0.1 ansible_connection=local

Tester la connexion :

    ansible all -m ping

Si tout est OK, tu verras pong.