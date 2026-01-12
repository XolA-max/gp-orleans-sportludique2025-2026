# Inventaire GLPI - Linux

Inventorier les appareils Linux sur GLPI.

---

## 1. Prérequis

Avant de commencer, assurez-vous d'avoir :

- ✅ GLPI installé - [Guide d'installation de GLPI](GLPI.md)
- ✅ Ansible installé - [Guide d'installation d'Ansible](../Ansible/Ansible.md)
- ✅ Accès SSH aux différents services (authentification + mot de passe)

---

---

## 2. Activer l'inventaire dans GLPI

Dans **l'interface web** de GLPI :

> **Configuration** → **Inventory** → **Enable inventory**

!!! info "Important"
    Cette étape est cruciale pour permettre à GLPI de recevoir les inventaires des agents.

---

---

## 3. Création du playbook d'installation

Sur le serveur Ansible, créer le fichier :

!!! info
    `sudo nano /etc/ansible/playbooks/install_glpi_agent.yml`

    ```yaml
    ---
    - name: Installer et configurer l'agent GLPI sur toutes les VM Linux
      hosts: all_linux
      become: yes
    
      vars:
        glpi_release_folder: "1.7"
        glpi_agent_filename: "glpi-agent_1.7-1_all.deb"
        glpi_agent_url: "https://github.com/glpi-project/glpi-agent/releases/download/{{ glpi_release_folder }}/{{ glpi_agent_filename}}"
        glpi_server_url: "http://172.28.120.8:80/front/inventory.php"
    
      tasks:
    
        - name: 1. Mettre à jour le cache APT
          ansible.builtin.apt:
            update_cache: yes
          tags: update
    
        - name: 2. Télécharger le package GLPI Agent (DEB)
          ansible.builtin.get_url:
            url: "{{ glpi_agent_url }}"
            dest: "/tmp/glpi-agent.deb"
            mode: '0644'
    
        - name: 3. Installer le package GLPI Agent
          ansible.builtin.apt:
            deb: "/tmp/glpi-agent.deb"
            state: present
            update_cache: yes
          notify:
            - Démarrer et Activer GLPI Agent
    
        # --- TÂCHES DE NETTOYAGE ET DE CONFIGURATION (NO AUTH) ---
    
        - name: 4a. Configurer l'URL du serveur GLPI dans agent.cfg
          ansible.builtin.lineinfile:
            path: /etc/glpi-agent/agent.cfg
            regexp: '^server ='
            line: 'server = {{ glpi_server_url }}'
            state: present
          notify: Redémarrer GLPI Agent pour appliquer la config
    
        - name: 4b. Supprimer l'ancienne configuration de login (pour Aucune Authentification)
          ansible.builtin.lineinfile:
            path: /etc/glpi-agent/agent.cfg
            regexp: '^user ='
            state: absent
          notify: Redémarrer GLPI Agent pour appliquer la config
    
        - name: 4c. Supprimer l'ancienne configuration de mot de passe
          ansible.builtin.lineinfile:
            path: /etc/glpi-agent/agent.cfg
            regexp: '^password ='
            state: absent
          notify: Redémarrer GLPI Agent pour appliquer la config
    
      handlers:
    
        # 1. Handler pour démarrer l'agent après l'installation
        - name: Démarrer et Activer GLPI Agent
          ansible.builtin.systemd:
            name: glpi-agent
            enabled: yes
            state: started
    
        # 2. Handler pour redémarrer l'agent après une modification de config
        - name: Redémarrer GLPI Agent pour appliquer la config
          ansible.builtin.systemd:
            name: glpi-agent
            enabled: yes
            state: restarted
    
    # --- Forcer l'ajout dans GLPI ---
    
    - name: Forcer et vérifier l'inventaire GLPI
      hosts: all_linux
      become: yes
      tasks:
    
        - name: Lancer l'inventaire GLPI
          ansible.builtin.command:
            cmd: glpi-agent --server http://192.168.140.225/ --force
          register: glpi_inventory
          changed_when: "'sending prolog request' in glpi_inventory.stdout"
          failed_when: glpi_inventory.rc != 0
    
        - name: Inventaire GLPI effectué
          ansible.builtin.debug:
            msg: "✅ Inventaire GLPI envoyé avec succès sur {{ inventory_hostname }}"
          when: glpi_inventory.changed
    
        - name: Inventaire GLPI déjà existant
          ansible.builtin.debug:
            msg: "ℹ️ Inventaire GLPI déjà présent pour {{ inventory_hostname }}"
          when: not glpi_inventory.changed
    ```

!!! info "Note"
    Ce playbook installe l'agent GLPI, le configure et force l'envoi de l'inventaire automatiquement.

---

---

## 4. Configuration du fichier d'inventaire Ansible

Éditer le fichier d'inventaire :

!!! info
    `sudo nano /etc/ansible/hosts`

    Ajouter tous les hôtes à inventorier (le groupe `[all_linux]` permet de cibler toutes ces machines simultanément) :

    ```ini
    [all_linux]
    # DNS
    ORL-DNS-REDIRECTEUR ansible_host=192.168.140.85 ansible_user=etudiant
    ORL-DNS-AUTORITE ansible_host=192.168.140.95 ansible_user=etudiant
    ORL-DNS-AUTORITE-SECOND ansible_host=192.168.140.125 ansible_user=etudiant
    ORL-DNS-REDIRECTEUR-SECOND ansible_host=192.168.140.135 ansible_user=etudiant
    
    # WEB
    ORL-SRV-WEB ansible_host=192.168.140.105 ansible_user=etudiant
    ORL-AUTORITE-CERT ansible_host=192.168.140.115 ansible_user=etudiant
    ORL-SRV-WORDPRESS ansible_host=192.168.140.145 ansible_user=etudiant
    ORL-REVERSE_PROXY ansible_host=192.168.140.155 ansible_user=etudiant
    ORL-REVERSE_PROXY-SECOND ansible_host=192.168.140.215 ansible_user=etudiant
    
    # SERVEUR
    ORL-SRV-BDD ansible_host=192.168.140.165 ansible_user=etudiant
    ORL-SRV-DOCKER ansible_host=192.168.140.225 ansible_user=etudiant
    ORL-SRV-GRAYLOG ansible_host=192.168.140.235 ansible_user=etudiant
    ORL-SRV-ANSIBLE ansible_host=192.168.140.240 ansible_user=etudiant
    ```

!!! info "Note"
    Adaptez les adresses IP et les noms d'utilisateur selon votre infrastructure.

---

---

## 5. Création d'une clé d'authentification SSH

### Pourquoi une clé SSH ?

Il n'est pas possible d'utiliser une authentification SSH par mot de passe, car le serveur Ansible ne connaît pas les mots de passe des machines distantes. Il est donc nécessaire de mettre en place une **authentification par clé publique**.

### Générer la paire de clés

Sur le serveur Ansible :

!!! info
    ```bash
    ssh-keygen -t ed25519
    ```
    
    *   Appuyez sur **Entrée** pour accepter l'emplacement par défaut.
    *   **Ne définissez PAS de passphrase** (appuyez sur Entrée).

!!! info "Note"
    Aucune passphrase n'est définie car le serveur Ansible ne pourrait pas la saisir automatiquement. Cela permet des connexions SSH non interactives, indispensables au bon fonctionnement d'Ansible.

---

---

## 6. Copie de la clé publique sur les serveurs

La clé publique générée doit être **copiée** sur l'ensemble des **machines virtuelles** à administrer et à inventorier.

### 6.1 Copie manuelle (méthode longue)

!!! info
    ```bash
    ssh-copy-id utilisateur@192.168.X.X
    ```

### 6.2 Script de copie automatisée (méthode rapide)

Créer un script pour copier automatiquement la clé sur toutes les machines :

!!! info
    `nano copy-ssh-keys.sh`

    ```bash
    #!/bin/bash
    
    INVENTORY="/etc/ansible/hosts"
    
    while read line; do
      ip=$(echo "$line" | grep -oP 'ansible_host=\K[0-9.]+')
      user=$(echo "$line" | grep -oP 'ansible_user=\K\S+')
      
      if [ -n "$ip" ] && [ -n "$user" ]; then
        echo "Copie de la clé vers $user@$ip"
        ssh-copy-id "$user@$ip"
      fi
    done < "$INVENTORY"
    ```

    Exécuter le script :
    ```bash
    chmod +x copy-ssh-keys.sh
    ./copy-ssh-keys.sh
    ```

!!! info "Note"
    Le script va demander le mot de passe SSH pour chaque machine. Entrez le mot de passe lorsque demandé.
    Une fois cette étape réalisée, le serveur Ansible pourra se connecter automatiquement aux VM sans demander de mot de passe.

### 6.3 Vérification de la connexion SSH

!!! info
    Tester la connexion sans mot de passe :
    ```bash
    ansible all_linux -m ping
    ```

---

---

## 7. Lancer le playbook d'installation des agents

### Exécution du playbook

!!! info
    ```bash
    ansible-playbook /etc/ansible/playbooks/install_glpi_agent.yml --ask-become-pass
    ```

    *   **ansible-playbook** : Lance l'exécution du playbook.
    *   **--ask-become-pass** : Demande le mot de passe **sudo** de l'utilisateur distant.

### Déroulement de l'installation

Le playbook va automatiquement :
1. ✅ Mettre à jour le cache APT
2. ✅ Télécharger l'agent GLPI
3. ✅ Installer l'agent GLPI
4. ✅ Configurer l'URL du serveur GLPI
5. ✅ Démarrer et activer le service
6. ✅ Forcer l'envoi de l'inventaire à GLPI

---

---

## 8. Vérification dans GLPI

### Accéder à l'interface GLPI

*   URL : `http://IP_SERVEUR_GLPI`

### Vérifier les ordinateurs inventoriés

*   Aller dans : **Parc** → **Ordinateurs**

Vous devriez voir apparaître tous vos serveurs Linux avec leurs informations détaillées (Nom d'hôte, Adresse IP, OS, Hardware, etc.).

!!! info "Note"
    L'inventaire se met à jour automatiquement selon la configuration de l'agent (par défaut toutes les 24 heures).

---

---

## 9. Forcer l'inventaire manuellement (optionnel)

### Sur une machine spécifique

!!! info
    ```bash
    sudo glpi-agent --server http://192.168.140.225/ --force
    ```

### Sur toutes les machines via Ansible

!!! info
    ```bash
    ansible all_linux -m command -a "glpi-agent --server http://192.168.140.225/ --force" --become --ask-become-pass
    ```

---

---

## 10. Dépannage

### Problème : L'agent ne se connecte pas à GLPI

!!! warning "Diagnostic"
    Vérifier le statut de l'agent :
    ```bash
    systemctl status glpi-agent
    ```

    Vérifier les logs :
    ```bash
    journalctl -u glpi-agent -f
    ```

    Vérifier la configuration :
    ```bash
    cat /etc/glpi-agent/agent.cfg
    ```

### Problème : Les machines n'apparaissent pas dans GLPI

!!! warning "Diagnostic"
    1.  Vérifier que l'inventaire est activé dans GLPI (`Configuration` → `Inventory` → `Enable inventory`).
    2.  Forcer l'inventaire manuellement.
    3.  Vérifier que le serveur GLPI est accessible :
        ```bash
        curl http://IP_GLPI/front/inventory.php
        ```

### Problème : Permission denied lors de l'exécution du playbook

!!! warning "Diagnostic"
    1.  S'assurer que la clé SSH a bien été copiée :
        ```bash
        ssh etudiant@192.168.140.85
        ```
    2.  Vérifier que l'utilisateur a les droits sudo :
        ```bash
        ssh etudiant@192.168.140.85 "sudo whoami"
        ```

---

---

## 11. Maintenance et mise à jour

### Mettre à jour l'agent GLPI

!!! info
    Modifier la version dans le playbook :
    ```yaml
    vars:
      glpi_release_folder: "1.8"
      glpi_agent_filename: "glpi-agent_1.8-1_all.deb"
    ```

    Relancer le playbook :
    ```bash
    ansible-playbook /etc/ansible/playbooks/install_glpi_agent.yml --ask-become-pass
    ```

### Désinstaller l'agent GLPI

!!! info
    ```bash
    ansible all_linux -m apt -a "name=glpi-agent state=absent" --become --ask-become-pass
    ```

---

---

## 12. Bonnes pratiques

### Sécurité
- ✅ Restreindre l'accès SSH au serveur Ansible uniquement.
- ✅ Utiliser des clés SSH avec passphrase en production.
- ✅ Limiter les droits sudo de l'utilisateur Ansible.
- ✅ Activer l'authentification dans GLPI pour l'inventaire.

### Organisation
- ✅ Documenter les modifications dans le playbook.
- ✅ Tester sur un environnement de test avant la production.
- ✅ Sauvegarder régulièrement la configuration GLPI.
- ✅ Maintenir l'inventaire Ansible à jour.

### Monitoring
- ✅ Surveiller les logs de l'agent GLPI.
- ✅ Vérifier régulièrement que tous les agents envoient leur inventaire.
- ✅ Mettre en place des alertes en cas d'échec d'inventaire.

---

---

## Ressources utiles

- [Documentation GLPI Agent](https://glpi-agent.readthedocs.io)
- [GitHub GLPI Agent](https://github.com/glpi-project/glpi-agent)
- [Documentation Ansible](https://docs.ansible.com)
- [Forum GLPI](https://forum.glpi-project.org)

---

*Dernière mise à jour : Décembre 2024*