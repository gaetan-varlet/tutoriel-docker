# Docker

## Play with Docker

*Play with Docker* permet de prendre en main Docker sans l'installer

## Les première commandes

```bash
docker run --rm bash echo "Salut"
```
détails de la commande :
- `docker` est le nom de la commande qui permet de donner des ordres à Docker
- `run` est l'action que l'on demande d'exécuter à Docker, ici de créer et lancer un conteneur
- `--rm` est un paramètre optionel qui dit à Docker de supprimer le conteneur quand il aura fini de s'exécuter
- `bash` est le nom de l'image
- `echo "Salut"` est la commande a exécuter dans le conteneur

Docker cherche l'image de bash localement dans son cash et exécute la commande s'il trouve l'image dans son cash, sinon il télécharge l'image sur le registre **Docker Hub** avant d'exécuter la commande


```bash
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
bash                latest              d97778c0b68a        3 weeks ago         15.1MB
```
Cela donne la liste des images présentes dans le cash de notre serveur
- `REPOSITORY` est le nom du dépôt d'où provient l'image : si l'image provient d'un dépôt officiel de docker hub, il y a juste le nom de l'image. Si l'image provient d'un dépôt public, il y a le nom du compte de l'utilisateur suivi de `/` puis du nom de l'image
- `TAG` permet de connaître la version de l'image
- `IMAGE ID`est un hash de l'image

Pour utiliser une version spécifique d'une image, il faut préciser la version après le nom de l'image séparé de deux points : `bash:3.2`


```bash
docker run -ti --rm bash
bash-5.0#
```
- `-ti` : `-t` dit à Docker qu'on souhaite avoir accès à un terminal  en pseudo TTY, c'est-à-dire avec l'entrée standard (clavier) et la sortie standard (écran). `-i` démarre le conteneur en mode interactif. On va interagir avec le conteneur au clavier via le pseudo TTY que l'on a créé.
- on se retrouve ensuite dans le conteneur
- `exit` permet de sortir du conteneur et revenir sur la machine hôte

```bash
docker ps
CONTAINER ID    IMAGE   COMMAND     CREATED     STATUS      PORTS   NAMES
```
- liste les conteneurs créés sur notre machine
- `CONTAINER ID` id unique du conteneur
- `IMAGE` nom de l'image utilisée
- `COMMAND` : commande utilisé au lancement du conteneur
- `CREATED` : date de création du conteneur
- `STATUS` : état du conteneur
- `PORTS` : on verra plus tard
- `NAMES` : nom créé automatiquement en mettant deux noms au hasard à la suite


## Les IDs et les raccourcis

- les IDs sont sont identifiant hexadécimaux uniques nécessaires pour manipuler les **objets** Docker (Images, Conteneurs, etc...)
- il y a possible de faire de la complétion des commandes Docker avec `Tab`
- `docker history` permet d'afficher la liste des couches composant une image

## Préserver les données

