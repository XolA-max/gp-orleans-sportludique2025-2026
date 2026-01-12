# Guide utilisation Github

---

## 1. Prérequis

### Installation de Git

!!! info
    ```bash
    sudo apt-get install git
    ```

### Installation de Python + pip

!!! info
    ```bash
    sudo apt-get install python3-pip
    ```

### Installation de MkDocs

!!! info
    ```bash
    pip install mkdocs
    ```

---

---

## 2. Configuration de Python et de MkDocs

### 2.1 Création et activation d’un environnement virtuel

!!! info
    ```bash
    python3 -m venv venv
    source venv/bin/activate
    ```
    Le prompt doit afficher : `(venv)`

    Pour installer les dépendances :
    ```bash
    pip install -r requirements.txt
    ```

### 2.2 Installation globale (optionnelle)

!!! info
    ```bash
    sudo python3 -m pip install --upgrade pip
    sudo python3 -m pip install mkdocs mkdocs-material
    ```

    Vérification :
    ```bash
    which mkdocs
    ```
    
    Résultat attendu : `/usr/local/bin/mkdocs`

---

---

## 3. Créer un nouveau dépôt GitHub

1.  Connecte-toi sur [github.com](https://github.com).
2.  Clique sur **New repository**.
3.  Remplis le formulaire :
    *   **Repository name** : `mon-projet-mkdocs`
    *   **Visibility** : Public
    *   Coche **Add a README file**
4.  Clique sur **Create repository**.

---

---

## 4. Configurer les clés SSH sur GitHub

1.  Aller dans **Settings** > **SSH and GPG keys**.
2.  Cliquer **New SSH key**.
3.  Coller la clé SSH publique générée :
    ```bash
    cat ~/.ssh/id_rsa.pub
    ```
4.  Sauvegarder.

---

---

## 5. Cloner le dépôt sur ta machine

Sur ton PC :

!!! info
    ```bash
    cd /chemin/de/mon/projet
    git clone git@github.com:ton-utilisateur/mon-projet-mkdocs.git
    ```

---

---

## 6. Ajouter les fichiers du projet

Une fois le projet créé localement :

1.  Ajouter ton code (`src/`).
2.  Ajouter la doc (`docs/`).
3.  Ajouter un `.gitignore`.
4.  Ajouter le `mkdocs.yml`.

Ensuite :

!!! info "Commit & Push"
    ```bash
    git add .
    git commit -m "Initialisation du projet" 
    git push origin main
    ```

---

---

## 7. Créer une branche (bonnes pratiques GitHub)

Dans le terminal :

!!! info "Création de branche"
    ```bash
    cd /chemin/de/mon/projet
    git checkout -b NOM_BRANCHE
    ```

Après modifications :

!!! info "Envoi des modifications"
    ```bash
    git add .
    git commit -m "Ajout fonctionnalité X"
    git push origin NOM_BRANCHE
    ```

---

---

## 8. Créer une Pull Request (PR)

Dans GitHub :

1.  Aller dans l’onglet **Pull Requests**.
2.  Cliquer **New pull request**.
3.  Comparer :
    *   **base** : `main`
    *   **compare** : `NOM_BRANCHE`
4.  Cliquer **Create pull request**.
5.  Ajouter description et commentaires.
6.  Le propriétaire valide ou demande des modifications.

---

---

## 9. Ajouter un workflow CI/CD dans GitHub Actions

1.  Aller dans **Actions**.
2.  Choisir un modèle : "Deploy static content to GitHub Pages" (ou "Set up a workflow yourself").
3.  GitHub crée un fichier : `.github/workflows/ci.yml`.
4.  Modifier le workflow dans l’éditeur GitHub (ex : installer MkDocs, générer, déployer).
5.  Faire **Commit changes**.

---

---

## 10. Synchroniser ton dépôt local après modification du workflow

Toujours dans ton projet local :

!!! info
    ```bash
    cd /chemin/de/mon/projet
    git pull origin main
    ```

---

---

## 11. Activer GitHub Pages

1.  Aller dans **Settings** -> **Pages** -> **Build and Deployment**.
2.  **Source** : `Deploy from a branch`
3.  **Branch** : `gh-deploy` (ou celle configurée dans ton workflow).
4.  Cliquer **Save**.

GitHub affiche alors l’URL du site :
`https://ton-utilisateur.github.io/mon-projet-mkdocs/`
