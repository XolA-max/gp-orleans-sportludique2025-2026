# Github
## 1. Prérequis

---

### Instalation de Git
```
    sudo apt-get install git
```
---

###  Instalation de Python + pip
```
    sudo apt-get install python3-pip
```

---

###  Instalation de MkDocs
```
    pip install mkdocs
```

---

---

## 2. Configuration de Python et de MkDocs
### 2.1 Création et activation d’un environnement virtuel
```
    python3 -m venv venv
    source venv/bin/activate
```
   Le prompt doit afficher : ```(venv)```

Pour installer vos dépendances :
```
    pip install -r requirements.txt
```
---
### 2.2 Installation globale (optionnelle)
```
sudo python3 -m pip install --upgrade pip
sudo python3 -m pip install mkdocs mkdocs-material
```
Vérification :
```
which mkdocs
```
Résultat attendu :
```/usr/local/bin/mkdocs```

---

---

## 3. Créer un nouveau dépôt GitHub 

Connecte-toi sur https://github.com. Créer un nouveau dépôt GitHub

    Clique sur New repository

    Remplis :

        Repository name : mon-projet-mkdocs

        Description (optionnel)

        Visibility : Public

    Coche Add a README file

    Clique Create repository

---

---

 ## 4. Configurer les clés SSH sur GitHub 

    Aller dans Settings > SSH and GPG keys

    Cliquer New SSH key

    Coller la clé SSH publique générée :

    cat ~/.ssh/id_rsa.pub

    Sauvegarder

---

---

## 5. Cloner le dépôt sur ta machine

Sur ton PC :
```
cd /chemin/de/mon/projet
``` 
```
git clone git@github.com:ton-utilisateur/mon-projet-mkdocs.git
```

---

---

## 6. Ajouter les fichiers du projet

Une fois le projet créé localement :

    Ajouter ton code (src/)

    Ajouter la doc (docs/)

    Ajouter un .gitignore

    Ajouter le mkdocs.yml

Ensuite :
``` 
git add .
git commit -m "Initialisation du projet" 
git push origin main
``` 
---

---

 ## 7. Créer une branche (bonnes pratiques GitHub)

Dans le terminal :
``` 
cd /chemin/de/mon/projet

git checkout -b NOM_BRANCHE
``` 
Après modifications :
``` 
git add .
git commit -m "Ajout fonctionnalité X"
git push 
``` 
---

---

## 8. Créer une Pull Request (PR)

Dans GitHub :

    Aller dans l’onglet Pull Requests

        Cliquer New pull request

            Comparer :

                     base : main

                     compare : NOM_BRANCHE

            Cliquer Create pull request

    Ajouter description et commentaires

    Le propriétaire valide ou demande des modifications

---

---

## 9. Ajouter un workflow CI/CD dans GitHub Actions

    Aller dans Actions
            Choisir un modèle : "Deploy static content to GitHub Pages"
            → ou cliquer Set up a workflow yourself



 GitHub crée un fichier :
  ```   
 .github/workflows/ci.yml 
 ```

    Modifier le workflow dans l’éditeur GitHub (ex : installer MkDocs, générer, déployer)

    Faire Commit changes

---

---

## 10. Synchroniser ton dépôt local après modification du workflow

Toujours dans ton projet local :

```
cd /chemin/de/mon/projet
git pull origin main
```
---

---

## 11. Activer GitHub Pages

    Aller dans -> [Settings] -> [Pages] -> [Build and Deployment]

            Source :
                Branch : gh-deploy (ou celle configurée dans ton workflow)

    Cliquer Save

GitHub affiche alors l’URL du site :

``` https://ton-utilisateur.github.io/mon-projet-mkdocs/```
