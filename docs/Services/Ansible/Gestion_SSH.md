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

!!! info "Note"
    La méthode manuelle est principalement destinée au premier serveur (hôte maître) ou en cas de problème ponctuel. Privilégiez l'automatisation.

#### Création de l'utilisateur

On crée l'utilisateur `ansible`. Le mot de passe peut être laissé vide ou verrouillé, car l'authentification se fera exclusivement par clé SSH.

```bash
sudo adduser ansible
```

!!! info "💡 Note sur l'utilisateur pivot"
    Si la machine cible utilise l'utilisateur `admin` au lieu de `etudiant`, il faut modifier la variable `ansible_user` dans l'inventaire avant de lancer le playbook de secours.

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

!!! danger "Crucial"
    Les permissions `chmod 700` sur le dossier `.ssh` et `chmod 600` sur `authorized_keys` sont **cruciales**. Si elles sont trop permissives (ex: 777), le service SSH refusera par défaut la connexion par mesure de sécurité.

## 2. Sécurisation SSH (Le fichier `sshd_config`)

Il est recommandé de durcir la configuration SSH dans le fichier `/etc/ssh/sshd_config`.

!!! warning "Attention"
    Avant de désactiver `PasswordAuthentication`, assurez-vous impérativement que l'authentification par clé fonctionne correctement sur chaque machine, pour éviter d'être bloqué hors du système.

| Paramètre | Valeur Sécurisée | Usage de Secours |
| --- | --- | --- |
| `PasswordAuthentication` | `no` | `yes` (pour dépanner) |
| `PubkeyAuthentication` | `yes` | `yes` |
| `KbdInteractiveAuthentication` | `no` | `yes` (si le mot de passe ne passe pas) |

Après modification, n'oubliez pas de redémarrer le service SSH :

!!! danger "Attention"
    Toujours tester la connexion dans un nouvel onglet avant de redémarrer le service SSH, pour éviter de perdre définitivement l'accès à la VM.

```bash
sudo systemctl restart ssh
```

## 3. Automatisation de la Sécurisation SSH

Une fois l'accès initial garanti, un script permet de configurer le service SSH sur l'ensemble du parc informatique. Ce playbook applique les durcissements recommandés en désactivant la connexion par mot de passe et en déployant les clés publiques nécessaires.

### Préparation : Le dossier `keys/`

Avant d'exécuter l'automatisation de la sécurisation, il est nécessaire de créer un répertoire contenant les clés publiques de chaque administrateur. 

!!! warning "Important"
    Les noms des fichiers `.pub` doivent correspondre exactement aux pseudonymes définis dans le `loop` du playbook. L'extension `.pub` est indispensable.

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

## 4. Dépannage et Remise en état

### Dépannage : Erreurs Communes

Voici les erreurs les plus fréquemment rencontrées et comment les résoudre :

- **L'erreur Permission denied** : Même si l'utilisateur existe, si le dossier `~/.ssh` n'a pas les droits `700` et si le fichier `authorized_keys` n'a pas les droits `600`, SSH rejettera systématiquement la connexion par mesure de sécurité.
- **Le piège du propriétaire** : Si le dossier `.ssh` appartient à `root` au lieu de l'utilisateur concerné (par exemple `ansible`), l'authentification par clé SSH ne fonctionnera jamais.
- **L'erreur de désérialisation JSON** : Si Ansible a besoin d'un mot de passe pour `sudo` mais qu'il est lancé sans l'option `--ask-become-pass` (ou `-K`), il plantera en affichant une erreur confuse de désérialisation JSON.

### Le Mode Debug (Port 2222)

La commande magique pour voir ce qui se passe "sous le capot" :

```bash
sudo /usr/sbin/sshd -d -p 2222
```
C'est grâce à ça qu'on a compris que le problème venait de la configuration et non du mot de passe lui-même.

### Procédure de remise en état des accès

Le playbook de secours `fix_ansible_access.yml` qu'on a fait ensemble est crucial. C'est lui qui permet de réparer 15 machines d'un coup quand elles passent en `UNREACHABLE`.

*(Exemple de structure type)*
```yaml
---
- name: "Dépannage : Remise en état des accès Ansible"
  hosts: all
  become: yes
  
  tasks:
    - name: "S'assurer que le dossier .ssh appartient à ansible avec les bons droits"
      ansible.builtin.file:
        path: /home/ansible/.ssh
        state: directory
        owner: ansible
        group: ansible
        mode: '0700'

    - name: "Déployer la clé publique et corriger les droits (600)"
      ansible.posix.authorized_key:
        user: ansible
        path: /home/ansible/.ssh/authorized_keys
        key: "{{ lookup('file', 'keys/ansible.pub') }}"
        state: present

    - name: "Rétablir les droits SUDO sans mot de passe"
      ansible.builtin.copy:
        content: "ansible ALL=(ALL) NOPASSWD: ALL"
        dest: /etc/sudoers.d/ansible
        mode: '0440'
```

Pour exécuter ce playbook sur les machines inaccessibles, il est crucial d'utiliser l'utilisateur pivot pour lequel on a encore un accès par mot de passe :

```bash
# Lancement du secours en utilisant l'utilisateur pivot (ex: etudiant)
ansible-playbook fix_ansible_access.yml -u etudiant -k -K
```

*Rappel : `-k` pour le mot de passe SSH, `-K` pour le mot de passe sudo.*
