# installation de shibolet

télécharger shibbolet pour windwos et lancer l'installation en cochant connecter le a IIS meme si nous allons le connecter a apache. 

copier le ficheir d'exemple de configuration apache24.conf depuis le reprtoire de shibbolet C:\opt\shibboleth-sp\etc\shibboleth (par défaut) dans le repertoire C:\Apache24\conf\extra 

modifier le httpd.conf pour rajouter un include extra/apache24.conf 

modifiier le fichier shibbolet2.xml pour configurer la conenction à l'Idp et lister les chmaps a positionner dans REMOTE_USER

retirer le tag acl dans la section  <Handler type="Status" Location="/Status" />

modifier entityID dans <ApplicationDefaults entityID= pour mettre l'url du site a proteger

modifier REMOTE_USER pour mette email en premier

modifier attribute-map.xml pour rajouter l'attribut email venant de Azure AD

ouvrir l'url https://site.web/Status pour voir les info de shibbolet s'afficher

configurer Azure AD app

envoyer aux admin de Azure App  les url 

En reponse configuration de shibbolet avec les url de l'idp azure AD 

dans shibbolte2.xml section Metadatadriver et SSO