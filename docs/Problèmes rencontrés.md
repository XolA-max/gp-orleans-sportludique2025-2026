## Séance 03/09/25

PROBLEMES 

- Enorme latence sur le fait d’écrire sur les terminaux
- les lettres ne correspondaient pas et il faisait que des erreur

SOLUTIONS

- avoir pris un pc à part câble bien tendu (pas de solutions précise )
- connexions ssh à distance depuis Mon ordinateur

## Séance 08/09/25

PROBLEMES 

- liaison Trunck entre switch et le pc
- DNS mauvaise configuration 
création d’un nouveau DNS suppression de DNS initial qui étais relier au compte Administrateur 
Suppression de la VM

SOLUTIONS

- arrêter d’essayer une liaison entre les deux appareils
- ne pas supprimer le DNS principal et bien le configurer du premier coup 
`/!\`    `Cloner la VM pour ne pas avoir à recommencer l’installation`

## Séance 09/09/25

| Problème | Solution |
| --- | --- |
| Liaison interconnexion entre le switch et le routeur 
Le VLAN Créer (249) | créer le VLAN 249 et le `mettre en mode Access sur une Interface pour L’ACTIVER` Sinon il n’est pas actif et ne peux pas être lier à une liaison TRUNCK
conf t 

$code :$  `Vlan 249 description Interco` |
| ACL Problème de mauvaise acl  |  |
|  |  |
|  |  |
Test

