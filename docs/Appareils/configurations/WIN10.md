# Guide Windows 10 Prêt à l'emplois ou (PAE)

--- 

## Liaison avec Active Directory

#### Étape 1 : Configuration du serveur DNS

Configurez le serveur DNS de l'Active Directory dans les paramètres réseau de la machine.

1. Ouvrir les **Paramètres réseau**
2. Accéder aux propriétés de la carte réseau active
3. Configurer les **Propriétés IPv4**
4. Saisir l'adresse IP du serveur DNS de l'Active Directory

#### Étape 2 : Accès aux paramètres de domaine

Naviguez vers les paramètres de jonction de domaine :

1. Ouvrir **Système**
2. Sélectionner **Informations système**
3. Dans **Liens connexes**, cliquer sur **Domaine ou groupe de travail**
4. Cliquer sur **Modifier**

#### Étape 3 : Configuration du domaine

Dans la fenêtre de modification :

1. Saisir le **nom de la machine** (NetBIOS)
2. Sélectionner l'option **Domaine**
3. Saisir le **nom de domaine** complet (FQDN)
4. Cliquer sur **OK**

#### Étape 4 : Authentification

Lors de la demande d'authentification :

1. Saisir les identifiants d'un **compte utilisateur** présent dans l'Active Directory
2. Ce compte doit disposer des droits nécessaires pour joindre des machines au domaine
3. Valider l'authentification

#### Étape 5 : Finalisation

1. **Redémarrer** l'ordinateur lorsque le système le demande
2. Lors du redémarrage, s'identifier avec un compte du domaine
3. Sélectionner le domaine souhaité dans la liste déroulante (si plusieurs domaines disponibles)

### Vérification

Pour vérifier que la jonction au domaine a réussi :

- Ouvrir **Système** et vérifier que le nom de domaine apparaît
- Tester la connexion avec un compte utilisateur du domaine


### Dépannage

Si la jonction échoue, vérifier :

- La connectivité réseau avec le contrôleur de domaine
- La résolution DNS (test ping vers le nom de domaine)
- Les droits du compte utilisateur utilisé
- Les paramètres du pare-feu

