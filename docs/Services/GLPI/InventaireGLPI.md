# Inventorier les appareils Linux sur GLPI
## 1. Prérequis 
-  GLPI Installé [Guide d'installation de GLPI](gp-orleans-sportludique2025-2026/docs/Services/GLPI/GLPI.md)
-  Ansible Installé [Guide d'installation d'Ansible](gp-orleans-sportludique2025-2026/docs/Services/Ansible/Ansible.md)
-  Acces en SSH au differents services (authentification+ Mot de passe)  
### 2. Enable GLPI Inventory

-> Configuration 
    -> Inventory 
        -> enable inventory

## Sur le serveur Ansible :

## 3. Fichier /etc/ansible/playbooks/install_glpi_agent.yml
```
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

    # 2. Handler pour redémarrer l'agent après une modification de config (celui qui manquait)
    - name: Redémarrer GLPI Agent pour appliquer la config
      ansible.builtin.systemd:
        name: glpi-agent
        enabled: yes
        state: restarted

# --- Forcer l'ajout dans GLPI 

- name: Forcer et vérifier l'inventaire GLPI
  hosts: all_linux
  become: yes
  tasks:

    - name: Lancer l’inventaire GLPI
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

## fichier /etc/ansible/hosts
```                         
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
----
 
## 4. Création d'une clé d'authentification 
Il n’est pas possible d’utiliser une authentification SSH par mot de passe, car le serveur Ansible ne connaît pas les mots de passe des machines distantes. Il est donc nécessaire de mettre en place une authentification par clé publique.

Pour cela, nous générons une paire de clés SSH sur le serveur Ansible :
```
  ssh-keygen -t ed25519
```

Aucune passphrase n’est définie lors de la génération de la clé, car le serveur Ansible ne pourrait pas la saisir automatiquement. Cela permet ainsi des connexions SSH non interactives, indispensables au bon fonctionnement d’Ansible.
  

## 5. Copie de la clé publique sur les serveurs de l’infrastructure

La clé publique générée sur le serveur Ansible doit être **copiée** sur l’ensemble des **machines virtuelles** à **administrer** et à **inventorier**.
 Cette opération permet d’établir une authentification SSH par clé entre le serveur Ansible et les VM cibles.

### 5.1 La copie de la clé publique 
s’effectue à l’aide de la commande suivante :
```
  ssh-copy-id utilisateur@192.168.*.*
```

L’utilisateur spécifié doit être identique à celui défini par la variable `ansible_user` dans le fichier d’inventaire Ansible :

  
Cette commande doit être exécutée pour chaque machine virtuelle de l’infrastructure, **une par une**, y compris lorsque le nombre de serveurs est élevé. 

> Alors on utilise un **Script** pour nous **simplifier** la tâche
### 5.2 Script pour la copie de masse sur Machines
```
while read line; do   ip=$(echo "$line" | grep -oP 'ansible_host=\K[0-9.]+');   user=$(echo "$line" | grep -oP 'ansible_user=\K\S+');   if [ -n "$ip" ] && [ -n "$user" ]; then     echo "Copie de la clé vers $user@$ip";     ssh-copy-id "$user@$ip";   fi; done < "$INVENTORY"

```

> Une fois cette étape réalisée, le serveur Ansible pourra se connecter automatiquement aux VM sans demander de mot de passe.


---

## 6. Lancer le playbook d'installation des agents

   ansible-playbook /etc/ansible/playbooks/install_glpi_agent.yml --ask-become-pass
> on met le --ask-become-pass

## forcer l'inventaire des agents sur GLPI manuellement
> Sur la VM ciblée (exemple serveur Web) on effectue cette commande 

    sudo glpi-agent --server http://192.168.140.225/ --force


Cette commande doit être exécutée pour chaque machine virtuelle de l’infrastructure, **une par une**, y compris lorsque le nombre de serveurs est élevé. 


> c'est pourquoi on l'a déja mis dans le fichier install_glpi_agent.yml