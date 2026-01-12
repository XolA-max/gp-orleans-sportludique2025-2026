# Installation de Docker

---

## 1. Installation sur Debian/Ubuntu

### Mise à jour et dépendances

!!! info
    ```bash
    sudo apt-get update
    sudo apt-get install ca-certificates curl gnupg
    ```

### Ajout du dépôt Docker

!!! info
    ```bash
    sudo install -m 0755 -d /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    sudo chmod a+r /etc/apt/keyrings/docker.gpg
    
    echo \
      "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
      "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
      sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```

### Installation des paquets

!!! info
    ```bash
    sudo apt-get update
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```

### Vérification

!!! info
    ```bash
    sudo docker run hello-world
    ```

---

---

## 2. Commandes Utiles

| Commande | Description |
| :--- | :--- |
| `docker ps` | Liste les conteneurs actifs |
| `docker images` | Liste les images téléchargées |
| `docker system prune` | Nettoye les éléments inutilisés |