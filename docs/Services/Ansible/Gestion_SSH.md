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

Et si le script précédent ne fonctionne pas, il faudra donc créer l'utilisateur `ansible` à la main sur chaque VM cible.

!!! info "Note"
    La méthode manuelle est principalement destinée au premier serveur (hôte maître) ou en cas de problème ponctuel. Privilégiez toujours l'automatisation.

---

#### Étape 1 — Création de l'utilisateur `ansible`

On crée l'utilisateur `ansible`. Le mot de passe peut être laissé vide ou verrouillé, car l'authentification se fera exclusivement par clé SSH.

```bash
sudo adduser ansible
```

!!! info "💡 Note sur l'utilisateur pivot"
    Si la machine cible utilise l'utilisateur `admin` au lieu de `etudiant`, il faut adapter les chemins dans les commandes suivantes.

---

#### Étape 2 — Attribution des privilèges sudo (NOPASSWD)

Ansible a besoin d'exécuter des commandes root (`become: yes` dans les playbooks). Pour éviter qu'il soit bloqué par un prompt de mot de passe, on ajoute une règle **NOPASSWD** dans `/etc/sudoers.d/` :

```bash
echo "ansible ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ansible
sudo chmod 0440 /etc/sudoers.d/ansible
```

> **Pourquoi `chmod 0440` ?** Le service `sudo` vérifie que les fichiers inclus dans `/etc/sudoers.d/` ne sont pas modifiables par n'importe qui. Si les permissions sont trop ouvertes, `sudo` ignorera le fichier silencieusement. Le mode `0440` (lecture seule pour root et le groupe root) est le minimum requis.

---

#### Étape 3 — Préparation du répertoire SSH (utilisateur courant)

On prépare d'abord le dossier `.ssh` et le fichier `authorized_keys` sur l'utilisateur courant (par ex. `etudiant`) :

```bash
mkdir -p ~/.ssh
touch ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

> **Pourquoi ces commandes ?**
>
> - `mkdir -p ~/.ssh` : crée le dossier `.ssh` s'il n'existe pas encore (l'option `-p` empêche une erreur s'il existe déjà).
> - `touch ~/.ssh/authorized_keys` : crée le fichier `authorized_keys` vide s'il n'existe pas.
> - `chmod 700 ~/.ssh` : seul le propriétaire peut lire, écrire et traverser ce dossier. SSH exige cette restriction stricte, sinon il refusera l'authentification par clé.
> - `chmod 600 ~/.ssh/authorized_keys` : seul le propriétaire peut lire et écrire ce fichier. Même logique de sécurité que pour le dossier.

---

#### Étape 4 — Ajout des clés publiques autorisées

Ouvrez le fichier `authorized_keys` avec `nano` et collez-y les clés publiques de chaque administrateur et du serveur Ansible :

```bash
nano ~/.ssh/authorized_keys
```

Contenu à ajouter (une clé par ligne) :

```text
ssh-ed25519 AAAAC3...jkcsm etudiant@ORL-SRV-ANSIBLE
ssh-ed25519 AAAAC3...NhGC  admin-cyriak
ssh-ed25519 AAAAC3...JhPC  Admin-Louis
ssh-ed25519 AAAAC3...r6oG  antoine.mesange@gmail.fr
```

!!! warning "Clés masquées"
    Les clés ci-dessus sont volontairement tronquées pour cette documentation publique. Les clés complètes sont disponibles sur le serveur Ansible (`/etc/ansible/playbooks/keys/`) ou auprès de chaque administrateur.

Enregistrez (`Ctrl+O`, `Entrée`) et quittez (`Ctrl+X`).

---

#### Étape 5 — Copie et sécurisation du `.ssh` vers l'utilisateur `ansible`

Maintenant que les clés sont en place sur l'utilisateur courant, on copie tout le dossier `.ssh` vers le home de l'utilisateur `ansible` et on corrige les propriétaires et permissions :

```bash
# Créer le dossier .ssh pour ansible (au cas où il n'existerait pas)
sudo mkdir -p /home/ansible/.ssh

# Copier les clés autorisées de l'utilisateur courant vers ansible
sudo cp /home/etudiant/.ssh/authorized_keys /home/ansible/.ssh/authorized_keys

# Attribuer la propriété du dossier .ssh à l'utilisateur ansible
sudo chown -R ansible:ansible /home/ansible/.ssh

# Appliquer les permissions strictes exigées par SSH
sudo chmod 700 /home/ansible/.ssh
sudo chmod 600 /home/ansible/.ssh/authorized_keys
```

> **Pourquoi chacune de ces commandes est nécessaire ?**
>
> - `sudo mkdir -p /home/ansible/.ssh` : l'utilisateur `ansible` vient d'être créé, il n'a pas encore de dossier `.ssh`. On le crée manuellement.
> - `sudo cp ... authorized_keys` : on réutilise les clés déjà configurées pour l'utilisateur courant. Cela évite de devoir les recopier une par une.
> - `sudo chown -R ansible:ansible /home/ansible/.ssh` : **étape critique** — si le dossier `.ssh` ou `authorized_keys` appartient à `root` ou à un autre utilisateur, SSH refusera systématiquement la connexion par clé. Le propriétaire **doit** être `ansible`.
> - `sudo chmod 700 /home/ansible/.ssh` : le dossier `.ssh` ne doit être accessible qu'à son propriétaire. C'est une vérification de sécurité du daemon SSH (mode `StrictModes` activé par défaut).
> - `sudo chmod 600 /home/ansible/.ssh/authorized_keys` : même logique — le fichier des clés ne doit être lisible et modifiable que par l'utilisateur `ansible`.

!!! danger "Crucial"
    Les permissions `chmod 700` sur le dossier `.ssh` et `chmod 600` sur `authorized_keys` sont **cruciales**. De plus, le **propriétaire** doit impérativement être l'utilisateur `ansible` (et non `root`). Si l'une de ces conditions n'est pas remplie, SSH refusera la connexion par clé sans message d'erreur explicite.

## 2. Sécurisation SSH (Le fichier `sshd_config`)

Il est recommandé de durcir la configuration SSH dans le fichier `/etc/ssh/sshd_config`.

!!! warning "Attention"
    Avant de désactiver `PasswordAuthentication`, assurez-vous impérativement que l'authentification par clé fonctionne correctement sur chaque machine, pour éviter d'être bloqué hors du système.

| Paramètre | Valeur Sécurisée | Usage de Secours |
| --- | --- | --- |
| `PasswordAuthentication` | `no` | `yes` (pour dépanner) |
| `PubkeyAuthentication` | `yes` | `yes` |
| `KbdInteractiveAuthentication` | `no` | `yes` (si le mot de passe ne passe pas) |
| `ListenAddress` | `192.168.140.x` (IP management) | `0.0.0.0` (écoute sur toutes les interfaces) |

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

Ce playbook effectue trois actions de sécurisation sur l'ensemble du parc :

1. **Déploiement des clés SSH** : il lit les clés publiques du dossier `keys/` et les ajoute dans le fichier `authorized_keys` de l'utilisateur `ansible`.
2. **Désactivation de l'authentification par mot de passe** : il force `PasswordAuthentication no` dans `sshd_config`.
3. **Restriction réseau** : il configure `ListenAddress` pour que SSH n'écoute que sur l'interface du réseau de management (`192.168.140.0/24`), rendant les VM inaccessibles par SSH depuis les autres réseaux (clients, DMZ, etc.).

```yaml
---
- name: Securisation SSH de toute l'infrastructure
  hosts: all
  become: yes
  tasks:
    - name: "Ajout des cles publiques dans authorized_keys"
      ansible.posix.authorized_key:
        user: ansible
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

    - name: "Restriction : SSH ecoute uniquement sur l'interface 192.168.140.0/24"
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?ListenAddress'
        line: "ListenAddress {{ ansible_all_ipv4_addresses | select('match', '^192\\.168\\.140\\.') | first }}"
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
