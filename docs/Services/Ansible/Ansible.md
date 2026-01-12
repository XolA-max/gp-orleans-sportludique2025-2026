# Guide d'installation Ansible

---

## Installation

Depuis une machine Linux / Debian, mettre à jour le système :

!!! info "Mise à jour"
    ```bash
    sudo apt update
    sudo apt upgrade -y
    ```

Installer Ansible :

!!! info "Installation"
    ```bash
    sudo apt install ansible -y
    ```

---

---

## Configuration de Base (Test)

### Création du fichier d’inventaire

!!! info
    Création du répertoire et du fichier hosts :
    
    ```bash
    sudo mkdir -p /etc/ansible
    sudo nano /etc/ansible/hosts
    ```

    Contenu d'exemple pour tester sur la machine locale :
    
    ```ini
    [local]
    127.0.0.1 ansible_connection=local
    ```

### Test de connexion

!!! info
    Lancer un ping Ansible :
    
    ```bash
    ansible all -m ping
    ```
    
    Si tout est OK, le retour sera `pong`.
