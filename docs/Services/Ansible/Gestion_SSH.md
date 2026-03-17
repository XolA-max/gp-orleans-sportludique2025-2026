# Gestion des accès SSH et Automatisation avec Ansible

Cette documentation détaille la mise en place d'un accès sécurisé pour l'utilisateur de gestion Ansible, la configuration du service SSH, ainsi que les procédures d'automatisation et de dépannage.

## 1. Création de l'utilisateur de gestion (ansible)

Pour administrer les serveurs avec Ansible, il est nécessaire de créer un utilisateur dédié possèdant les droits d'administration sans interruption. L'approche recommandée est d'utiliser un playbook pour automatiser ce déploiement sur les machines.

### Méthode Automatisée (Recommandée) : `/etc/ansible/playbooks/add_ansible_user.yml`

Ce playbook crée l'utilisateur `ansible`, génère la configuration sudo et déploie la clé SSH pour permettre la gestion à distance. Créez-le dans le gestionnaire Ansible :

```yaml
---
- name: "Création de l'utilisateur ansible"
  hosts: all
  become: true

  vars:
    admin_user_name: ansible
    admin_user_ssh_public_key: "cle.pub ansible"

  tasks:

    - name: "Déterminer le groupe sudo"
      ansible.builtin.set_fact:
        _sudo_group: "sudo"

    - name: "Créer l'utilisateur"
      ansible.builtin.user:
        name: "{{ admin_user_name }}"
        shell: /bin/bash
        groups: "{{ _sudo_group }}"
        append: true
        password: "!"
        password_lock: true
        create_home: true

    - name: "Créer le dossier .ssh"
      ansible.builtin.file:
        path: "/home/{{ admin_user_name }}/.ssh"
        state: directory
        owner: "{{ admin_user_name }}"
        group: "{{ admin_user_name }}"
        mode: "0700"

    - name: "Déployer la clé publique"
      ansible.posix.authorized_key:
        user: "{{ admin_user_name }}"
        path: "/home/{{ admin_user_name }}/.ssh/authorized_keys"
        key: "{{ admin_user_ssh_public_key }}"
        state: present

    - name: "Configurer sudoers"
      ansible.builtin.copy:
        content: "{{ admin_user_name }} ALL=(ALL:ALL) NOPASSWD: ALL\n"
        dest: "/etc/sudoers.d/{{ admin_user_name }}"
        owner: root
        group: root
        mode: "0440"
        validate: "/usr/sbin/visudo -cf %s"
```

### Méthode Manuelle (Secours)

Et si le script précédent ne fonctionne pas, il faudra donc créer l'utilisateur `ansible` à la main :

> [!NOTE]
> La méthode manuelle est principalement destinée au premier serveur (hôte maître) ou en cas de problème ponctuel. Privilégiez l'automatisation.

#### Création de l'utilisateur

On crée l'utilisateur `ansible`. Le mot de passe peut être laissé vide ou verrouillé, car l'authentification se fera exclusivement par clé SSH.

```bash
sudo adduser ansible
```

#### Attribution des privilèges sudo

Pour permettre à Ansible d'exécuter des commandes en tant qu'administrateur sans demande de mot de passe, il faut créer un fichier de configuration dans `/etc/sudoers.d/`.

```bash
echo "ansible ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ansible
sudo chmod 440 /etc/sudoers.d/ansible
```

#### Configuration de l'identité SSH

Afin de permettre la connexion, la clé publique du serveur maître Ansible doit être copiée sur les hôtes cibles.

1. **Création de la clé SSH (sur le serveur maître)** :
   Si vous ne l'avez pas encore créée, générez une clé SSH :
   ```bash
   ssh-keygen -t ed25519 -C "nom_de_la_machine"
   ```
   Dès qu'elle est créée, utilisez la commande `cat` pour afficher son contenu, puis copiez tout le texte affiché :
   ```bash
   cat ~/.ssh/id_ed25519.pub
   ```

2. **Création du répertoire `.ssh` (sur l'hôte cible)** :
   Préparez le répertoire sur le serveur de destination :
   ```bash
   mkdir -p /home/etudiant/.ssh
   chmod 700 /home/etudiant/.ssh
   ```

3. **Ajout de la clé publique dans le fichier `authorized_keys`** :
   Ouvrez le fichier des clés autorisées avec l'éditeur `nano` :
   ```bash
   nano /home/etudiant/.ssh/authorized_keys
   ```
   *Collez-y le contenu copié précédemment.* Enregistrez (`Ctrl+O`, `Entrée`) et quittez (`Ctrl+X`).

   N'oubliez pas d'appliquer les permissions strictes sur le fichier :
   ```bash
   chmod 600 /home/etudiant/.ssh/authorized_keys
   ```

> [!CAUTION]
> Les permissions `chmod 700` sur le dossier `.ssh` et `chmod 600` sur `authorized_keys` sont **cruciales**. Si elles sont trop permissives (ex: 777), le service SSH refusera par défaut la connexion par mesure de sécurité.

## 2. Sécurisation SSH (Le fichier `sshd_config`)

Il est recommandé de durcir la configuration SSH dans le fichier `/etc/ssh/sshd_config`.

> [!WARNING]
> Avant de désactiver `PasswordAuthentication`, assurez-vous impérativement que l'authentification par clé fonctionne correctement sur chaque machine, pour éviter d'être bloqué hors du système.

| Paramètre | Valeur Sécurisée | Usage de Secours |
| --- | --- | --- |
| `PasswordAuthentication` | `no` | `yes` (pour dépanner) |
| `PubkeyAuthentication` | `yes` | `yes` |
| `KbdInteractiveAuthentication` | `no` | `yes` (si le mot de passe ne passe pas) |

Après modification, n'oubliez pas de redémarrer le service SSH :
```bash
sudo systemctl restart ssh
```

## 3. Automatisation de la Sécurisation SSH

Une fois l'accès initial garanti, un script permet de configurer le service SSH sur l'ensemble du parc informatique. Ce playbook applique les durcissements recommandés en désactivant la connexion par mot de passe et en déployant les clés publiques nécessaires.

### Préparation : Le dossier `keys/`

Avant d'exécuter l'automatisation de la sécurisation, il est nécessaire de créer un répertoire contenant les clés publiques de chaque administrateur. 

> [!IMPORTANT]
> Les noms des fichiers `.pub` doivent correspondre exactement aux pseudonymes définis dans le `loop` du playbook. L'extension `.pub` est indispensable.

1. Créez un répertoire `keys` dans le dossier des playbooks Ansible :
   ```bash
   mkdir -p /etc/ansible/playbooks/keys
   ```
2. Ajoutez un fichier par utilisateur contenant sa clé publique (`.pub`). Les noms de fichiers doivent correspondre aux pseudonymes utilisés dans le playbook :
   - `/etc/ansible/playbooks/keys/cyriak.pub`
   - `/etc/ansible/playbooks/keys/louis.pub`
   - `/etc/ansible/playbooks/keys/antoine.pub`
   - `/etc/ansible/playbooks/keys/ansible.pub`

### Le Playbook (`/etc/ansible/playbooks/setup_ssh.yml`)

Ce playbook utilise une boucle (`loop`) pour lire les clés SSH précédemment créées dans le dossier `keys/` et les transférer sur l'ensemble des machines gérées. Ensuite, il modifie le fichier de configuration SSH `sshd_config`.

```yaml
---
- name: Securisation SSH de toute l'infrastructure
  hosts: all
  become: yes
  tasks:
    - name: "Ajout des cles publiques dans authorized_keys"
      ansible.posix.authorized_key:
        user: etudiant
        state: present
        key: "{{ lookup('file', 'keys/' + item + '.pub') }}"
      loop:
        - cyriak
        - louis
        - antoine
        - ansible
        
    - name: "Verrouillage : Desactivation des mots de passe"
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?PasswordAuthentication'
        line: 'PasswordAuthentication no'
      notify: Restart SSH

  handlers:
    - name: Restart SSH
      service:
        name: ssh
        state: restarted
```
