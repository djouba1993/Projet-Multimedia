# Projet-Multimedia
### Schéma d'architecture du projet
Dans cet infra, nous avons mis en place un Sonarr comme librairie de séries qui utilise jackett comme indexeur.
il passe par transmission pour télécharger les séries qui seront stockées par la suite dans un médiathèque jellyfin.

![diagramme](https://user-images.githubusercontent.com/48188335/114879964-ef302d80-9e01-11eb-946e-624e62504e2d.png)

### Sonar 
**Sonarr** permet de rechercher vos fichiers .torrent et d’automatiser le téléchargement de vos séries préférées. Vous ajoutez une série dans l’interface en précisant la qualité et la langue souhaitées et Sonarr recherchera celle-ci via les indexers configurés. Enfin Sonarr ajoutera automatiquement dans transmission la série en téléchargement. Parmi les nombreuses fonctionnalités de Sonarr, celui-ci dispose de tâches automatiques et quotidiennes ajoutant automatiquement un épisode fraichement sorti. L’interface dispose aussi d’un calendrier répertoriant les sorties des prochains épisodes.

![Screenshot_208](https://user-images.githubusercontent.com/48188335/115953537-60e83580-a4ec-11eb-8aaf-cca7489fb7f3.png)


**Radarr** a un fonctionnement similaire à Sonarr mais  pour les films.

Cependant Sonarr et Radarr ne proposent que très peu d’indexers (ou trackers) aujourd’hui. Jackett permet de combler ce manque et prend en charge plus d’une centaine de trackers.
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






