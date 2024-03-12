# Installation Apache 2.4 comme reverse proxy SSL pour Lotus Domino sous Windows

## Introduction

Suite à différentes attaques sur notre site web, nous avons été contraints de procéder à une mise à niveau des technologies de cryptage SSL et d'authentification utilisées. Notre site, basé sur Lotus Domino, est considéré comme l'un des plus gros site Domino au monde. Malheureusement, la gestion des versions récentes de cryptage SSL est devenue de plus en plus compliquée au sein d'un serveur Domino, dont la maintenance a été soit arrêtée, soit fortement ralentie depuis son abandon par IBM. Pour pallier à ces difficultés, j'ai opté pour l'installation d'un serveur Apache en tant que reverse proxy devant notre serveur Domino, afin de gérer ces technologies de cryptage les plus récentes.

## Installation Apache

 Télécharger apache du site [Apache VS17 binaries and modules download](https://www.apachelounge.com/download/)

Installez-le dans C:\Apache24, le répertoire par défaut pour les fichiers de configuration d'Apache.

Dézippez l'archive dans ce répertoire et lancez la commande d'installation du service Windows :

```
C:\Apache24\bin>httpd.exe -k install
```

Si Apache est installé dans un répertoire différent, suivez ce tutoriel pour modifier les fichiers de configuration

[Apache installation version 2.4 Windows — EjnTricks](http://www.jouvinio.net/wiki/index.php/Apache_installation_version_2.4_Windows)

## Génération et certification des clés SSL

Pour générer une clé SSL pour Apache, exécutez la commande suivante sous Windows avec Git Bash

(https://gitforwindows.org/) :

```
` openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /C/Apache24/conf/monsite_com.key -out /C/Apache24/conf/monsite_com.crt`
```

Répondez aux questions, en veillant à bien indiquer le nom de domaine de votre site web pour le "Common Name".



> Generating a RSA private key
> .....+++++
> ..+++++
> writing new private key to 'C:/Apache24/conf/monsite_com.key'
> 
> -----
> 
> You are about to be asked to enter information that will be incorporated
> into your certificate request.
> What you are about to enter is what is called a Distinguished Name or a DN.
> There are quite a few fields but you can leave some blank
> For some fields there will be a default value,
> If you enter '.', the field will be left blank.
> 
> -----
> 
> Country Name (2 letter code) [AU]:**FR**
> State or Province Name (full name) [Some-State]:**IDF**
> Locality Name (eg, city) []:**Paris**
> Organization Name (eg, company) [Internet Widgits Pty Ltd]:**GADZ**
> Organizational Unit Name (eg, section) []:
> Common Name (e.g. server FQDN or YOUR name) []:**monsite.com**
> Email Address []:
> `

Générez ensuite une CSR pour l'envoyer à l'autorité de certification :

```
openssl req -new -key /C/Apache24/conf/monsite_com.key -out /C/Apache24/conf/monsite_com.csr
```

Vous recevrez un fichier .cer avec le certificat signé.

## Configuration d'Apache pour la gestion SSL et l'intégration du certificat

Modifiez le fichier httpd.conf situé dans C:\Apache24\conf.

Décommentez la ligne :

`LoadModule ssl_module modules/mod_ssl.so`

Ajouter la ligne suivante à la fin du fichier

`Include conf/extra/monsite.conf`

Créez le fichier C:\Apache24\conf\extra\monsite.conf avec le contenu suivant : 

```
Listen 443
<VirtualHost monsite.com:443>
    ServerName monsite.com
    DocumentRoot "${SRVROOT}/monsite"
    # HTTPS
    SSLEngine on
    SSLCipherSuite ALL:!aNULL:!ADH:!eNULL:!LOW:!EXP:RC4+RSA:+HIGH:+MEDIUM:+SSLv3
    SSLCertificateFile "C:\Apache24\conf\monsite_com_cert.cer"
    SSLCertificateKeyFile "C:\Apache24\conf\monsite_com.key"

    # Activation de HTTP/2
    Protocols h2 h2c http/1.1

    # Reverse Proxy
    ProxyPreserveHost On
    ProxyRequests off
    RequestHeader set X-Forwarded-Proto https
    RequestHeader unset X-Forwarded-Host

    # Sécurité renforcée
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
    Header set X-XSS-Protection "1; mode=block"

    # Redirection des requêtes externes vers le port HTTP de Domino
    ProxyPass / http://monsite.com:165/
    ProxyPassReverse / http://monsite.com:165/

    #ERROR PAGE
    ErrorDocument 503 /error.html

</VirtualHost>
<VirtualHost monsite.com:80>
    ServerName monsite.com
    Redirect / https://monsite.com/
</VirtualHost>
```

Cette configuration installe et configure Apache comme un reverse proxy SSL pour Lotus Domino, renforçant ainsi la sécurité du site.

## Modification de la configuration Domino

Ouvrez le client administrateur Lotus et accédez au document de configuration ('Current Server Document') de votre serveur Domino. Suivez les étapes ci-dessous pour modifier la configuration des ports :

1. Naviguez vers l'onglet 'Ports'.

2. Sélectionnez le sous-onglet 'Internet Ports', puis allez à la section 'Web'.

3. Vérifiez et ajustez les paramètres suivants si nécessaire :
- TCPIP/Port Number : configurez-le sur 165.

- TCPIP/Port Status : choisissez 'Enabled'.

- SSL/Port Status : sélectionnez 'Disabled'.

Pour que les modifications prennent effet, procédez comme suit :

4. Rendez-vous dans l'onglet 'Server', sous-onglet 'Status', puis dans le menu 'Server Tasks'.
5. Faites un clic droit sur la tâche HTTP et sélectionnez 'Restart task' pour redémarrer le service HTTP. Cela permet d'appliquer les modifications effectuées.

Ces réglages assurent que le serveur Domino utilise le port TCP/IP 165 pour les connexions web, avec le SSL désactivé pour ce port.