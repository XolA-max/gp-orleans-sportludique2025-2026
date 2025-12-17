## 12. 📝 Documentation Markdown & MkDocs

Cette section explique **comment écrire en Markdown**, **personnaliser les thèmes**, et **ajouter des extensions MkDocs**.

---

## 12.1 🧾 Syntaxe Markdown (essentiel)

### Titres
```md
# Titre 1
## Titre 2
### Titre 3
```

### Texte
```md
**gras**
*italique*
~~barré~~
```

### Listes
```md
- Élément
- Élément
  - Sous-élément
```

```md
1. Premier
2. Deuxième
```

### Liens & images
```md
[Lien vers GitHub](https://github.com)

![Image](assets/image.png)
```

### Code
```md
`code inline`
```

```md
```python
print("Hello World")
```
```

### Tableaux
```md
| Nom | Description |
|-----|-------------|
| MkDocs | Générateur de doc |
```

---

## 12.2 📂 Organisation des fichiers Markdown

```txt
docs/
├── index.md
├── installation.md
├── utilisation.md
├── api.md
└── assets/
    └── images/
```

Dans `mkdocs.yml` :
```yaml
nav:
  - Accueil: index.md
  - Installation: installation.md
  - Utilisation: utilisation.md
  - API: api.md
```

---

## 12.3 🎨 Thèmes MkDocs

### Thème par défaut
```yaml
theme: mkdocs
```

### Thème Material (recommandé)
```yaml
theme:
  name: material
  language: fr
```

### Options Material
```yaml
theme:
  name: material
  features:
    - navigation.tabs
    - navigation.instant
    - content.code.copy
  palette:
    - scheme: default
      primary: indigo
      accent: indigo
```

---

## 12.4 🧩 Extensions Markdown (plugins)

### Extensions Markdown
```yaml
markdown_extensions:
  - admonition
  - toc:
      permalink: true
  - tables
  - codehilite
```

### Exemples d’extensions

#### Admonitions (notes, warnings)
```md
!!! note "Information"
    Ceci est une note importante

!!! warning
    Attention à cette action
```

#### Blocs repliables
```yaml
markdown_extensions:
  - pymdownx.details
```

```md
??? info "Détails"
    Contenu caché
```

---

## 12.5 🔌 Plugins MkDocs

### Plugins utiles
```yaml
plugins:
  - search
  - git-revision-date-localized
  - minify
```

Installation :
```bash
pip install mkdocs-git-revision-date-localized-plugin mkdocs-minify-plugin
```

---

## 12.6 🧪 Commandes MkDocs utiles

```bash
mkdocs new .
mkdocs serve     # serveur local http://127.0.0.1:8000
mkdocs build     # génération du site
```

---

## 12.7 🛠️ Bonnes pratiques documentation

- 1 page = 1 sujet
- Titres clairs et hiérarchisés
- Exemples concrets
- Captures d’écran dans `docs/assets/`
- Toujours tester avec `mkdocs serve`

---

## 12.8 🌍 Multilingue (option avancée)

```yaml
plugins:
  - i18n:
      default_language: fr
      languages:
        fr: Français
        en: English
```