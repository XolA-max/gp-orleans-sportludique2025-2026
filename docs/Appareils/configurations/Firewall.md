## FireWall physique

### Accès à l’interface Web d’administration a l'installation

```h

https://10.0.0.254

admin / admin
```
---

### Définir les interfaces réseau

Identifier les VLANs utilisés dans le réseau interne (IN) avec leur **nom**, **interface associée (IN)**, **numéro de VLAN**, ainsi que **l’adresse de management** accompagnée de son **masque de sous-réseau**.  
Répéter la même opération pour les autres VLANs présents sur l’infrastructure.  

Désactiver l’interface **IN** principale et **laisser uniquement les VLANs actifs**.  

Enfin, identifier l’adresse de sortie (vers l’extérieur) en précisant son **nom**, son **adresse IP** et son **masque de sous-réseau**.

---

### Navigation

Après avoir configuré les ports LAN et WAN, la configuration se fait depuis l’interface web de l’équipement avec l'addresse LAN de Mana.

---

### Définir les réseaux de destination

Passer en **mode écriture**.

Aller dans :  
`Objets → Réseaux → Type : Réseau → Ajouter`

Renseigner le **nom des réseaux distants** ainsi que **leurs adresses IP** et **leurs masques**.

---

### Définir les passerelles

Aller dans :  
`Objets → Réseaux → Type : Machine → Ajouter`

Attribuer un **nom** et renseigner **l’adresse IP de la passerelle** correspondante.

---

### Ajouter les routes de retour

Aller dans :  
`Réseau → Routage → Ajouter`

Renseigner le **réseau de destination**, **l’interface de sortie** et la **passerelle** associée.

---

### Régle de filtarge

|Protocol |	Source |	Port |	Destination |	Port |	Gateway |
|---------|--------|---------|--------------|--------|----------|
|*        |*       |*        |*             |*       |*         |

Cela permet de n’avoir aucun problème au niveau des règles de filtrage.


## Règles de Pare-feu Stormshield - Configuration Actuelle

| N° | État | Action | Source | Destination | Services | Protocole | Commentaire |
|----|------|--------|--------|-------------|----------|-----------|-------------|
| 1 | ON | Passer | Any | DNS-Autorité | dns | udp | Résolution DNS externe |
| 2 | ON | Passer | Any | DNS-A-P | dns | udp | DNS secondaire |
| 3 | ON | Passer | Any | Reverse-Proxy | http, https | tcp | Point d'entrée web public |
| 4 | ON | Passer | Any | Serv-Mail | imap, smtp | tcp | Services de messagerie |
| 5 | ON | Passer | WordPress | Serv-BDD-Wordpr | mysql | tcp | WordPress vers base de données |
| 6 | ON | Passer | Reverse-Proxy | Serv-Apache | http, https | tcp | Reverse proxy vers Apache |
| 7 | ON | Passer | Reverse-Proxy | WordPress | http, https | tcp | Reverse proxy vers WordPress |
| 8 | ON | Passer | clients, Serveurs, Network_VLAN_245 | Any | Any | - | Accès sortant réseau interne |
| 9 | ON | Passer | Network_VLAN_140 | Any | Any | - | Accès sortant VLAN 140 |
| 10 | ON | Bloquer | Any | Any | Any | - | **Règle de blocage par défaut** |

**Architecture :**
- **Accès publics** : DNS, Reverse-Proxy, Serveur Mail
- **Services internes** : Apache et WordPress (accessibles uniquement via Reverse-Proxy)
- **Base de données** : Accessible uniquement depuis WordPress
- **Sécurité** : Blocage par défaut de tout trafic non autorisé
---

### Desactiver le mode furtif

Aller dans :  
`Protection de sécurité → Protocoles → Protocoles IP → IP → Mode furtif`

---

## Configuration des Règles SSL

### Politique de sécurité : SSL Antoine

| N° | État | Action | URL - CN | Commentaire |
|----|------|--------|----------|-------------|
| 1 | ON | Bloquer sans déchiffrer | pornography | Blocage sites pornographiques |
| 2 | ON | Bloquer sans déchiffrer | online | Blocage sites en ligne |
| 3 | ON | Bloquer sans déchiffrer | illegal | Blocage sites illégaux |
| 4 | ON | Bloquer sans déchiffrer | warez | Blocage sites de téléchargement illégal |
| 5 | ON | **Bloquer sans déchiffrer** | **ads** | **Blocage publicités** |
| 6 | ON | Bloquer sans déchiffrer | bank | Blocage sites bancaires |
| 7 | ON | Bloquer sans déchiffrer | entertainment | Blocage sites de divertissement |
| 8 | OFF | Bloquer sans déchiffrer | shopping | Blocage sites d'achat (désactivé) |
| 9 | ON | Déchiffrer | Any | Règle par défaut - Déchiffrer tout |
| 10 | ON | **Passer sans déchiffrer** | **proxyssl_by...** | **Exceptions SSL - Ne pas déchiffrer certains serveurs** |

---

## Description des Actions

### Types d'actions disponibles

| Action | Icône | Description | Utilisation |
|--------|-------|-------------|-------------|
| **Bloquer sans déchiffrer** | 🚫 | Bloque l'accès au site sans inspecter le contenu | Catégories interdites (porn, warez, illegal) |
| **Déchiffrer** | 🔓 | Déchiffre le trafic SSL pour inspection | Inspection du trafic HTTPS autorisé |
| **Passer sans déchiffrer** | ✅ | Autorise le trafic sans déchiffrement | Sites sensibles (banques, santé, exceptions) |

---

## Catégories de Filtrage

### Catégories bloquées (actives)

| Catégorie | Description | Justification | État |
|-----------|-------------|---------------|------|
| **pornography** | Sites à contenu pornographique | Politique d'usage acceptable | ✅ Actif |
| **online** | Sites en ligne (générique) | Contrôle d'accès | ✅ Actif |
| **illegal** | Sites au contenu illégal | Conformité légale | ✅ Actif |
| **warez** | Téléchargement illégal de logiciels | Protection propriété intellectuelle | ✅ Actif |
| **ads** | Publicités et trackers | Protection et performance | ✅ Actif |
| **bank** | Sites bancaires | Sécurité (éviter MITM) | ✅ Actif |
| **entertainment** | Sites de divertissement | Productivité | ✅ Actif |

### Catégories désactivées

| Catégorie | Description | État |
|-----------|-------------|------|
| **shopping** | Sites de commerce en ligne | ❌ Désactivé |

---

## Règles Spéciales

### Règle 9 : Déchiffrement par défaut

---

## FireWall virtuel

OPNsense fonctionne sur une machine virtuelle Linux.

---

### Accès à l’interface Web d’administration

L’accès à l’interface web d’administration se fait grâce à l’adresse IP configurée lors de l’installation d’OPNsense.

---

### Définir les interfaces réseau

La configuration des interfaces réseau se fait en ligne de commande après l’installation.  
Il est important d’identifier correctement les adresses MAC et de les associer aux bonnes adresses IP, aussi bien pour les interfaces LAN (Management) que WAN.
Bien mettre l'addresse ip sans le masque ensuite mettre le masque cidr pour que opnsense puisse le configurer.

---

### Navigation

Après avoir configuré les ports LAN, WAN et DMZ public et privée, la configuration s’effectue depuis l’interface web de l’équipement, en utilisant l’adresse IP LAN du réseau Management.  
Il faut également attribuer une adresse IP à l’interface DMZ, correspondant au sous-réseau de la DMZ.

---

### Définir les passerelles

Sur les interfaces DMZ et WAN, il est nécessaire de renseigner les passerelles (gateway) en leur attribuant un **nom** et une **adresse IP** correspondante.

---

### Règles de filtrage

#### Regle Pass All
Appliquer les règles suivantes sur les interfaces WAN et DMZ :

| Protocole | Source | Port source | Destination | Port destination | Passerelle |
|------------|---------|--------------|--------------|------------------|-------------|
| *          | *       | *            | *            | *                | *           |

Cette configuration permet de ne rencontrer aucun blocage au niveau du filtrage, le temps de valider le bon fonctionnement du réseau.


#### Interface WAN

| N° | Protocol | Source | Port Source | Destination | Port Destination | Gateway | Schedule | Description |
|----|----------|--------|-------------|-------------|------------------|---------|----------|-------------|
| 1 | IPv4 UDP | 172.28.96.0/24 | * | * | 53 (DNS) | * | * | Vlan 240 -> DNS |
| 2 | IPv4 TCP | 172.28.96.0/24 | * | * | 80 (HTTP) | * | * | Vlan 240 -> HTTP |
| 3 | IPv4 TCP | 172.28.96.0/24 | * | * | 443 (HTTPS) | * | * | Vlan 240 -> HTTPS |
| 4 | IPv4 TCP | 172.28.96.0/24 | * | * | 25 (SMTP) | * | * | Vlan 240 -> SMTP |
| 5 | IPv4 TCP | 172.28.96.0/24 | * | * | 143 (IMAP) | * | * | Vlan 240 -> IMAP |
| 6 | IPv4 UDP | 172.28.120.0/24 | * | * | 53 (DNS) | * | * | Vlan 241 -> DNS |
| 7 | IPv4 TCP | 172.28.120.0/24 | * | * | 80 (HTTP) | * | * | Vlan 241 -> HTTP |
| 8 | IPv4 TCP | 172.28.120.0/24 | * | * | 443 (HTTPS) | * | * | Vlan 241 -> HTTPS |
| 9 | IPv4 TCP | 172.28.120.8 | * | 192.168.55.1 | 3306 | * | * | Serveur 120.8 -> BDD |
| 10 | IPv4 * | * | * | * | * | * | * | **Deny All (par défaut)** |

#### Interface DMZPRV

| N° | Protocol | Source | Port Source | Destination | Port Destination | Gateway | Schedule | Description |
|----|----------|--------|-------------|-------------|------------------|---------|----------|-------------|
| 1 | IPv4 * | * | * | * | * | * | * | **Deny All (par défaut)** |

#### Interface DMZ

| N° | Protocol | Source | Port Source | Destination | Port Destination | Gateway | Schedule | Description |
|----|----------|--------|-------------|-------------|------------------|---------|----------|-------------|
| 1 | IPv4 TCP | 192.168.45.6 | * | 192.168.55.1 | 3306 | * | * | DMZ -> BDD MySQL |
| 2 | IPv4 UDP | 192.168.45.0/24 | * | 172.28.120.3 | 53 (DNS) | * | * | DMZ -> DNS |
| 3 | IPv4 * | * | * | * | * | * | * | **Deny All (par défaut)** |

---

### Résumé des flux autorisés

#### Depuis WAN (réseaux internes)
- **172.28.96.0/24** : Accès DNS, HTTP, HTTPS, SMTP, IMAP vers Internet
- **172.28.120.0/24** : Accès DNS, HTTP, HTTPS vers Internet
- **172.28.120.8** : Accès MySQL vers 192.168.55.1 (serveur BDD)

#### Depuis DMZ (192.168.45.0/24)
- **192.168.45.6** : Accès MySQL vers 192.168.55.1 (serveur BDD)
- **192.168.45.0/24** : Résolution DNS vers 172.28.120.3

#### Depuis DMZPRV
- **Aucun flux autorisé** (tout bloqué par défaut)

#### Ajout d'une DMZ privée

### Vue d'ensemble
```
Internet
   |
[Stormshield] ← Pare-feu principal
   |
   ├─── DMZ Publique (existante)
   |    ├─ Reverse-Proxy
   |    ├─ Serv-Apache
   |    ├─ WordPress
   |    └─ Serv-Mail
   |
   └─── [OPNsense virtuel] 
        |
        └─── DMZ Privée (nouvelle)
             ├─ Serv-BDD-WordPress
   |
   └─── Reseaux interne
```

#### Segmentation réseau
| Zone | Réseau | Rôle | Pare-feu |
|------|--------|------|----------|
| DMZ Publique | 192.168.45.0/24 | Serveurs web exposés | Stormshield |
| DMZ Privée | 192.168.55.0/24 | Bases de données et backend | OPNsense |

---

