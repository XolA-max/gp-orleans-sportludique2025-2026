# Liaison GLPI à L'AD (LDAP)

## 1. Création d'une liaison 

- #### Configuration -> Authentification -> LDAP Directory -> nouveau

![Info-Vide](NewLDAP.png)
-
- Nom : Active Directory - it-connect.local
- Serveur par défaut : Oui
- Actif : Oui
- Serveur : 10.10.100.11
- Port : 389
- Filtre de connexion : (&(objectClass=user)(objectCategory=person)(!(userAccountControl:1.2.840.113556.1.4.803:=2)))
- BaseDN : DC=orl,DC=orleans,DC=sportludique,DC=fr
- Utiliser bind : Oui
- DN du compte : syncglpi@orl.orleans.sportludique.fr #nom utilisateur@nomdedomaine 
- Mot de passe du compte : Mot de passe du compte "Sync_GLPI"
- Champ de l'identifiant : userprincipalname #pas obligatoire
- Champ de synchronisation : objectguid #pas obligatoire
 



![Infos-Orleans](NewLDAPRempli.png)


## 2. Phase de Test LDAP
Après avoir remplis le LDAP Directory on test pour voir si il fonctionne correctement 

géneralement les 3 premières phases vont fonctionner correctement :
- TCP stream (flux tcp) 
- base DN 
- LDAP URI 
### Pour la phase Bind connection si vous avez l'erreur :

- #### Bind connection Authentication failed: Invalid credentials(49)
c'est que vous avez surement une erreur dans le champ root DN 
![alt text](NewLDAPErreurRootDN.png)


- #### Pour l'erreur Strong(er) authentication required (8)

GLPI veux utiliser LDAPS (version sécurisé de LDAP)
mais le DC ne l'autorise pas il faut alors le forcer à l'utiliser cliquer ici : [pour fixer l'erreur de LDAP](gp-orleans-sportludique2025-2026/docs/Services/Activedirectory/LDAP.md)
> /!\ utiliser LDAP est dangereux en effect il n'y a pas de chiffrement sur les données  /!\ et ne dois pas être mit en production comme ça ! 

#### Test réussi ! 

![alt text](NewLDAPtestok.png)


## 3.

