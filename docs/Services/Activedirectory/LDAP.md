# Principe et utilisation de LDAP

## Windows Serveur 2025

Depuis Windows Server 2019 (patch 2020) et renforcé dans Windows Server 2022 / 2025, les connexions LDAP simples non chiffrées (port 389) sont désormais bloquées ou limitées.
>Raison : Les identifiants sont envoyés en clair sur le réseau. Pas de Confidentialité

#### Erreurs typique :
```
    can't contact LDAP server 
```
```
    Strong(er) authentication required (8)
```

> Recommandation : Utiliser LDAPS (port 636) pour chiffrer les échanges entre Proxmox et Active Directory.

💡 Note : Le mode LDAP est temporaire. Une migration vers LDAPS sera effectuée pour sécuriser les échanges. Voici la configuration du LADPS qui a été réalisé : Tuto LDAPS

#### Méthode : via la Stratégie de groupe (GPO)

    Ouvrir la console de gestion des stratégies de groupe (GPMC).
    Éditer la GPO appliquée aux contrôleurs de domaine : default domain policy → Clique droit → Modifier.

#### Naviguer vers :

    Configuration ordinateur
    → Stratégies
      → Paramètres Windows
         → Paramètres de sécurité
            → Stratégies locales
               → Options de sécurité

#### Paramètres à modifier :

| paramètres | Valeur |
|-------|:----------:|
|Contrôleur de domaine : configuration requise pour le jeton de liaison du canal du serveur LDAP|Lorsqu’il est pris en charge
|Contrôleur de domaine : conditions requises pour la signature de serveur LDAP|Aucun|
|Contrôleur de domaine : application des conditions requises pour la signature de serveur LDAP|Désactivé|

####  Appliquer la GPO puis forcer la mise à jour :

    gpupdate /force
