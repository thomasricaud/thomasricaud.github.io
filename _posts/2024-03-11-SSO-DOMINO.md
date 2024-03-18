# **Configuration du SSO sur Domino : Activation des Headers de Connexion**

### Activer les Headers de Connexion sur votre serveur Domino

Pour améliorer l'intégration du SSO avec votre serveur Domino, suivez ces étapes :

1. **Modification du fichier `notes.ini`** : Ajoutez la ligne `HTTPEnableConnectorHeaders=1` au fichier `notes.ini` de votre serveur Domino. 

2. **Redémarrage de la tâche HTTP** : Afin de prendre en compte cette nouvelle configuration, un redémarrage de la tâche HTTP sur votre serveur est nécessaire. 

**Fonctionnement de cette configuration :**

Grâce à ce paramètre, la tâche HTTP de Domino pourra intercepter et utiliser le header HTTP `$WRSU`. Ce processus permet à Domino d'identifier de manière fiable et sécurisée le document person de l'utilisateur dans le `Names.nsf`, essentiel pour initialiser une session HTTP. Le header doit impérativement contenir la valeur du champ `FullName` de l'utilisateur, par exemple, `CN=username/O=Organization`.

### Securiser l'alimentation du Header

Pour sécuriser l'ajout d'informations dans les Headers HTTP qui sont envoyées au serveur Domino par une application externe (telle qu'Apache, IIS, ou IBM HTTP Server), et prévenir une usurpation d'identité, mettez en place d'un firewall .

Configurez le pour bloquer tout accès direct aux ports HTTP/HTTPS du serveur Domino venant de sources externes. Cela garantit que toutes les requêtes passent d'abord par l'application externe sécurisée.