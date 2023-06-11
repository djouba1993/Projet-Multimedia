# Projet-Multimedia
### Schéma d'architecture du projet
Dans cet infra, nous avons mis en place un Sonarr comme librairie de séries qui utilise jackett comme indexeur.
il passe par transmission pour télécharger les séries qui seront stockées par la suite dans un médiathèque jellyfin.

![diagramme](https://user-images.githubusercontent.com/48188335/114879964-ef302d80-9e01-11eb-946e-624e62504e2d.png)

### Sonar 
**Sonarr** permet de rechercher vos fichiers .torrent et d’automatiser le téléchargement de vos séries préférées. Vous ajoutez une série dans l’interface en précisant la qualité et la langue souhaitées et Sonarr recherchera celle-ci via les indexers configurés. Enfin Sonarr ajoutera automatiquement dans transmission la série en téléchargement. Parmi les nombreuses fonctionnalités de Sonarr, celui-ci dispose de tâches automatiques et quotidiennes ajoutant automatiquement un épisode fraichement sorti. L’interface dispose aussi d’un calendrier répertoriant les sorties des prochains épisodes.

![Screenshot_208](https://user-images.githubusercontent.com/48188335/115953537-60e83580-a4ec-11eb-8aaf-cca7489fb7f3.png)


**Radarr** a un fonctionnement similaire à Sonarr mais  pour les films.

Cependant **Sonarr** et **Radarr** ne proposent que très peu d’indexers (ou trackers) aujourd’hui. Jackett permet de combler ce manque et prend en charge plus d’une centaine de trackers.
Jackett fonctionne comme un serveur proxy, lorsque vous effectuez une recherche via Sonarr ou Radarr, celui-ci transforme et transmet la requête au tracker, analyse la réponse puis renvoie les résultats à l’application émettrice.

![Screenshot_209](https://user-images.githubusercontent.com/48188335/115953643-07ccd180-a4ed-11eb-834b-efd821205878.png)


### Mise à jour automatique des images Docker
Nous allons automatiser la mise à jour des images via une règle crontab. Celle-ci sera planifiée pour exécution tous les jours à 23h59. Les traces liées à l’exécution de ces commandes seront enregistrées dans un fichier /var/log/docker-updater.log.

**1. Editer la Crontab**
```
omar@ubuntu-server:~/projet-multimedia$ sudo crontab  -e
```
**2. Ajouter la ligne suivante et sauvargarder**

```
59 23 * * * (cd /home/omar/projet-multimedia && /usr/local/bin/docker-compose pull && /usr/local/bin/docker-compose up -d --remove-orphans && /usr/bin/docker image prune -f) > /var/log/docker-updater.log 2>&1
```
### Mise jour automatique les images de tous les fichiers docker-compose.yml et relancer tous les stacks
Comme vous pouvez le constater, mettre à jour une image nécessite de se rendre dans le répertoire de chaque docker-compose.yml et d’exécuter la commande de mise à jour et d’exécution de docker-compose. Nous allons automatiser cette tâche grâce à un script bash avec son fichier de configuration associé contenant l’ensemble des emplacements de nos fichiers docker-compose.yml. Nous allons créer une règle crontab où le script de mise à jour sera planifié pour exécution tous les jours à 23h59. Les traces liées à l’exécution du script seront enregistrées dans un fichier /var/log/docker-updater.log

**1. Créez le fichier /opt/docker-updater/docker-updater contenant les lignes suivantes:**
```
#!/bin/bash
# exec &> Sortie.log
# List of docker-compose configs
DCOMPOSE=containers-to-update.conf

# Update all docker compose scripts 
DIR_SCRIPT=`dirname $0`
if [ -e "$DIR_SCRIPT/$DCOMPOSE" ]
then
    cat "$DIR_SCRIPT/$DCOMPOSE" | /bin/grep -v '^#' | 
    while read conf
    do
  if [ -e "$conf" ]
  then
      dir=$(dirname "$conf")
      compfile=$(basename "$conf")    
      cd "$dir"
      /usr/local/bin/docker-compose pull && /usr/local/bin/docker-compose -f "$compfile" up -d --remove-orphans && /usr/bin/docker image prune -f 2>&1
  else
      echo "docker compose file $conf does not exist..."
  fi
    done
fi
```
**2. Ajoutez les droits d’exécution sur le script /opt/docker-updater/docker-updater:**
```
omar@ubuntu-server:/opt/docker-updater$ sudo chmod ugo+x /opt/docker-updater/docker-updater
```
**3. Créez le fichier de configuration containers-to-update.conf dans le répertoire /opt/docker-updater et ajoutez le path absolu de chaque fichier docker-compose.yml:**
```
omar@ubuntu-server:/opt/docker-updater$ cat containers-to-update.conf 
/home/omar/projet-multimedia/docker-compose.yml
/home/omar/projet-multimedia/traefik/docker-compose.yml
omar@ubuntu-server:/opt/docker-updater$
```
**4. Éditez la crontab:**
```
omar@ubuntu-server:~/projet-multimedia$ sudo crontab  -e
```
**5. Ajouter la ligne suivante et sauvargarder**
```
59 23 * * * /opt/docker-updater/docker-updater > /var/log/docker-updater.log 2>&1
```

### Traefik

**Traefik** est un reverse-proxy moderne conçu pour faciliter le déploiement des microservices (Docker, Kubernetes, AWS, etc.). Traefik est extrêmement simple à configurer et gère automatiquement vos certificats délivrés par Let’s Encrypt. De plus, il est capable de charger vos containers dynamiquement sans interruption de service et dispose d’un dashboard affichant l’ensemble de vos routes configurées.
ici on vous montre le dashbord de notre **traefik** avec tous les services qu'il faut.

![Screenshot_210](https://user-images.githubusercontent.com/48188335/115954291-66477f00-a4f0-11eb-8cc1-7b0d0ec799a2.png)

![Screenshot_211](https://user-images.githubusercontent.com/48188335/115954353-cd653380-a4f0-11eb-9d3b-ad84ab7ab8df.png)

![Screenshot_212](https://user-images.githubusercontent.com/48188335/115954401-1ddc9100-a4f1-11eb-82b2-8efa67ec143d.png)









