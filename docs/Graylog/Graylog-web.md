### 1. Interface Web

Se connecter a l'interface via :

```
Admin 
```

Mot de passe noter dans cette commande :

```
echo -n "MotDePasseAdmin" | sha256sum | cut -d' ' -f1
```

Le mot de passe sera donc MotDePasseAdmin dans cette situation :

### 1.1 Création d'un Input pour Syslog

Connectez-vous à l'interface de Graylog, cliquez sur "System" dans le menu puis sur "Inputs". Dans la liste déroulante, sélectionnez "Syslog UDP" puis cliquez sur le bouton intitulé "Launch new input". Il est également possible de créer un Input Syslog en TCP, mais cela implique d'utiliser un certificat TLS : c'est un plus pour la sécurité, mais ce point ne sera pas abordé dans cet article.

```
Un assistant va s'afficher à l'écran. Commencez par donner un nom à cet Input, par exemple "Graylog_UDP_Rsyslog_Linux" et choisissez un port. Par défaut, le port est "514" mais vous pouvez le personnaliser.
```

```
Le port 514 et déja actif sur le Graylog donc il est recommander de l'utiliser.
```

Vous pouvez aussi cocher l'option "Store full message" pour que le message de log complet soit stocké dans Graylog. Cliquez sur "Launch Input".

Le nouvel Input a été créé et il est désormais actif. Désormais, Graylog peut recevoir les logs Syslog sur le port 514/UDP. 

Pour cela voir si des logs apparaissent en temps réel sur le graphe du nouvel Input.

#### A. Créer un nouvel Index Linux

La création d'un Index dans Graylog permet de stocker les jouraux des machines. C'est une structure de stockage qui contient les logs reçus, Graylog utilise OpenSearch comme moteur de stockage et les messagessont indexes pour permettre une meileur fluidité, rapidités et efficacités.

À partir de Graylog, cliquez sur "System" dans le menu puis sur "Indices". Au sein de la nouvelle page qui s'affiche, cliquez sur "Create index set".

Nommez cet index, ajoutez une description et un préfixe, avant de valider. Ici nous allons stocker tous les journaux dans cet index. Il est également possibe de créer des index spécifiques afin de stocker uniquement certains logs comme (SSH).

#### B. Créer un Stream

Pour créer un nouveau Stream, cliquez sur "Streams" dans le menu principal de Graylog. Ensuite, cliquez sur le bouton "Create stream" situé sur la droite. Dans la fenêtre qui apparaît, nommez le stream et choisissez l'index que vous venez de nommez pour le champ nommé "Index Set". Validez.

Dans les paramètres du stream, cliquez sur le bouton "Add stream rule" pour ajouter une nouvelle règle de routage des messages. Si vous ne trouvez pas cette fenêtre, cliquez sur "Streams" dans le menu, puis sur la ligne correspondante à votre stream, cliquez sur "More" puis "Manage Rules".

Ensuite choissisez le type "match input" et selectionnez l'input Rsyslog en UDP crée précédemment. Validez avec le bouton "Create Rule".Ainsi tout les messages à destination de notre nouvel Input seront envoyés dans l'index.

Le nouveau stream doit s'afficher dans la liste et doit être en état "running". Si la bande passante indique "0msg/s" c'est normal aucun journal vers l'input Rsyslog UDP n'est envoyer. Pour voir les journaux d'un stream il suffit de cliquez simplement sur son nom.

##### Tout est prêt coté de Graylog. 

### 1.2 Installation et configuration de Rsyslog sur Graylog

#### A. Installer le paquet Rsyslog

Commencez par mettre à jour le cache des paquets et installer le paquet nommé "rsyslog".

```
sudo apt-get update
sudo apt-get install rsyslog
```
Puis, vérifiez le statut du service. En principal, il est déjà en cours d'exécution.

```
sudo systemctl status rsyslog
```

#### B. Configurer Rsyslog

Rsyslog dispose d'un fichier de configuration principal situé à cet emplacement :

```
/etc/rsyslog.conf
```

Le répertoire "/etc/rsyslog.d/" est utiliser pour stocker des fichiers de configuration supplémentaires pour Rsyslog. Dans le fichier de configuration principal, il y a une directive Include permettant d'importer tous les fichiers ".conf" situé ce répertoire. Ceci permet d'avoir des fichiers complémentaires pour configurer Rsyslog sans modifier le fichier principal.

Dans ce répertoire, nous allons créer le fichier intitulé "10-graylog.conf" :

```
sudo nano /etc/rsyslog.d/10-graylog.conf
```
Dans ce fichier, insérez cette ligne :

```
*.* @192.168.10.220:12514;RSYSLOG_SyslogProtocol23Format
```

Quand c'est fait, enregistrez le fichier et redémarrez Rsyslog.

```
sudo systemctl restart rsyslog.service
```

Suite à cette action, les premiers messages devraient arriver sur votre serveur Graylog !

### 1.3 Afficher les journaux Linux dans Graylog

À partir de Graylog, vous pouvez cliquer sur "Streams" et sélectionner votre nouveau stream pour afficher les messages associés. Sinon, cliquez sur "Search" et effectuez la sélection de votre Steam et lancez une recherche.

1 - Sélectionnez la période pour laquelle afficher les messages. Par défaut, ce sont les messages des 5 dernières minutes qui s'affichent.

2 - Sélectionnez le(s) stream(s) à afficher.

3 - Activer le refresh automatique de la liste des messages (toutes les 5 secondes, par exemple). Sinon, c'est statique et c'est à vous de faire un refresh manuel.

4 - Cliquez sur la loupe pour lancer la recherche après avoir modifié la période, le stream, ou le filtre.

5 - Barre de saisie pour indiquer vos filtres de recherche. Ici, je précise "source:srv\-docker" pour afficher uniquement les journaux de la nouvelle machine sur laquelle je viens de configurer Rsyslog.

Des journaux sont bien envoyés :