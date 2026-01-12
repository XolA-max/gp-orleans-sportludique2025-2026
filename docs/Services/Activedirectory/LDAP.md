# Principe et utilisation de LDAP

---

## Erreur liée à Windows Serveur 2025 pour LDAP

Depuis Windows Server 2019 (patch 2020) et renforcé dans Windows Server 2022 / 2025, les connexions LDAP simples non chiffrées (port 389) sont désormais bloquées ou limitées.

> Raison : Les identifiants sont envoyés en clair sur le réseau. Pas de Confidentialité.

### Erreurs typiques

!!! warning "Erreurs courantes"
    ```text
    can't contact LDAP server 
    ```
    
    ```text
    Strong(er) authentication required (8)
    ```

!!! info "Recommandation"
    Utiliser **LDAPS (port 636)** pour chiffrer les échanges entre Proxmox et Active Directory.

!!! note "Note"
    Le mode LDAP est temporaire. Une migration vers LDAPS sera effectuée pour sécuriser les échanges.

---

---

## Configuration GPO (Solution de contournement)

Si vous devez temporairement autoriser le LDAP simple, voici la méthode via GPO.

1.  Ouvrir la console de gestion des stratégies de groupe (GPMC).
2.  Éditer la GPO appliquée aux contrôleurs de domaine : **Default Domain Controllers Policy**.
3.  Naviguer vers :
    *   **Configuration ordinateur**
        *   **Stratégies**
            *   **Paramètres Windows**
                *   **Paramètres de sécurité**
                    *   **Stratégies locales**
                        *   **Options de sécurité**

### Paramètres à modifier

!!! info "Paramètres de Sécurité"
    | Paramètre | Valeur |
    | :--- | :---: |
    | Contrôleur de domaine : configuration requise pour le jeton de liaison du canal du serveur LDAP | **Lorsqu’il est pris en charge** |
    | Contrôleur de domaine : conditions requises pour la signature de serveur LDAP | **Aucun** |
    | Contrôleur de domaine : application des conditions requises pour la signature de serveur LDAP | **Désactivé** |

### Application

!!! info
    Appliquer la GPO puis forcer la mise à jour :
    
    ```powershell
    gpupdate /force
    ```
