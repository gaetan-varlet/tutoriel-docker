# Docker

## La virtualisation

La virtualisation permet d'émuler un ordinateur physique à l'aide d'un logiciel appelé hyperviseur, ce qui donne la possibilité de faire fonctionner un ou plusieurs ordinateurs virtuels sur un même ordinateur physique.

### Avant la virtualisation

Soit on installait toutes les applications sur un même ordinateur, soit on installait une application par ordinateur pour les séparer.

### La machine virtuelle (VM)

L'ordinateur qui accueille les VM est appelé hôte. Il contient un OS, des librairies et commandes, un **hyperviseur**.
Les VM, comme elles s'apparentent à un ordinateur physique, doivent aussi avoir leur OS, leurs librairies et commandes, une ou plusieurs applications.
L'inconvénient est que les VM doivent avoir des ressources dédiées (CPU, RAM) et l'OS qui prend de l'espace disque.

### Les hyperviseurs

ceux nécessitant un OS sur la machine hôte :

- **Virtualbox** (Oracle) et **QEMU** (open source) sont gratuits
- **VMWare** (VMWare) et **Virtual Server** (Microsoft) sont payants

ceux ne nécessitant pas d'OS sur la machine hôte (hyperviseurs BareMetal, qui intègre son propre OS minimal) :

- **Xen** (Open Source) et **LinuxKVM** (Open Source) sont gratuits
- **VMWare ESX** (VMWare) et **Hyper-V** (Microsoft) sont payants

hébergeurs de VMs dans le cloud :

- AWS (Amazon)
- Azure (Microsoft)
- Digital Ocean

### Les conteneurs

C'est une nouvelle solution de virtualisation. La machine hôte qui accueille les conteneurs est constitué des éléments suivants :

- un OS
- des librairies et commandes
- un **Container Engine**

![Schéma illustrant les différences entre une architecture avec VM vs Containers](vm-containers.png "Schéma illustrant les différences entre une architecture avec VM vs Containers")

Dans un conteneur, contrairement au conteu des VM, il n'y a pas d'OS. Les conteneurs utilisent l'OS de la machine hôte. Un conteneur est simplement un processus de la machine hôte. L'avantage par rapport à une architecture où l'on installe toutes ses applications sur la machine hôte est l'**isolation**. Le conteneur est isolé des autres processus tournant sur la machine hôte, sur lequel il n'a aucun accès.

Avantages des conteneurs :

- consommation réduite de CPU, RAM et de disque : les ressources ne sont pas allouées comme pour les VM mais **les ressources sont partagées**. Il est quand même possible de mettre des limites de consommation aux conteneurs
- possibilité de déployer davantage de conteneurs que de VM sur une machine hôte
- déploiement et démarrage très rapide

Nous allons étudier **Docker**, qui est un logiciel permettant de lancer des applications dans des conteneurs logiciels.

## Docker : les concepts de base

### Images et Conteneurs

Une image est un modèle pour un conteneur, qui est créé à partir d'une image. En POO, une image serait une classe et un conteneur une instance de cette image.
Les images sont conservés dans un registre. Dans un registre (registry), une image est placée dans un dépôt (repository) où l'on peut stocker différentes versions d'une image, que l'on identifie grâce à une étiquette (tag) qui sont souvent des numéros de version.
Si on fait une analogie avec Git, un registre est comme GitHub, un dépôt correspondrait à un projet.

### Layers

Les images sont constituées de couches (layers), chaque couche apportant des modifications à l'image pour former une nouvelle image. L'empilement de tous les calques forme l'image finale.
Il est possible de modifier une image pour l'adapter à nos besoins et produire une nouvelle image.

### Registres

Il existe un registre officiel administré par Docker, **Docker Hub**, qui référence plus d'un million d'images publiques. On y trouve des images officielles, mais aussi des images proposées par les membres de la communauté Docker.
Il est possible de déposer des images sur le registre public, ou en payant de disposer d'un registre privé.

### Play with Docker

*Play with Docker* permet de prendre en main Docker sans l'installer.

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

- `-ti` : `-t` dit à Docker qu'on souhaite avoir accès à un terminal en pseudo TTY, c'est-à-dire avec l'entrée standard (clavier) et la sortie standard (écran). `-i` démarre le conteneur en mode interactif. On va interagir avec le conteneur au clavier via le pseudo TTY que l'on a créé.
- on se retrouve ensuite dans le conteneur
- `exit` permet de sortir du conteneur et revenir sur la machine hôte

```bash
docker ps
CONTAINER ID    IMAGE   COMMAND     CREATED     STATUS      PORTS   NAMES
```

- liste les conteneurs créés sur notre machine (l'option `--all` permet de lister tous les conteneurs, par défaut uniquement les conteneurs actifs sont listés)
- `CONTAINER ID` id unique du conteneur
- `IMAGE` nom de l'image utilisée
- `COMMAND` : commande utilisé au lancement du conteneur
- `CREATED` : date de création du conteneur
- `STATUS` : état du conteneur
- `PORTS` : on verra plus tard
- `NAMES` : nom créé automatiquement en mettant deux noms au hasard à la suite

## Les IDs et les raccourcis

- les IDs sont des identifiant hexadécimaux uniques nécessaires pour manipuler les **objets** Docker (Images, Conteneurs, etc...)
- il est possible de faire de la complétion des commandes Docker avec `Tab`
- `docker history <image id>` permet d'afficher la liste des couches composant une image

## Préserver les données

- lorsque l'on fait des modifications dans un conteneur, par exemple écrire dans un fichier, ces modifications sont perdues avec la destruction du conteneur. **Les modifications dans un conteneur sont TEMPORAIRES**
- si on lance un conteneur sans l'option `--rm`, le conteneur ne sera pas détruit lorsqu'il aura fini de s'exécuter
- **docker run crée toujours un nouveau conteneur**. Pour relancer un conteneur arrêté, il faut utiliser la commande `docker start -ai <nom-du-conteneur>`. `-ai` est l'équivalent de `-ti` de la commande `docker run`. Les modifications faites dans le conteneur avant son arrêt sont toujours présentes car le conteneur n'a pas été détruit
- `docker stop` permet d'arrêter un conteneur de la même manière
- si la machine hôte est redémarrée, le **docker engine** sera aussi redémarré, seul resteront les conteneurs lancés sans l'option `--rm`

## Les volumes

Toutes les couches d'un conteneur, issues de l'image sont en lectures seules. Une couche en lecture/écriture est ajoutée à la création sur laquelle le conteneur travaille. Cette couche est dans le conteneur, elle n'est donc pas accessible depuis la machine hôte à cause du principe de cloisement. On ne peut donc pas sauvegarder de manière pérenne des données dans cette couche car elle disparaîtra quand le conteneur sera détruit.

La solution s'appelle **les volumes**. Un volume permet de monter un fichier ou un répertoire de la machine hôte directement dans un conteneur, ce qui permet d'écrire directement dans le système de fichiers de l'hôte et non plus dans la couche de travail du conteneur. Si le conteneur est supprimé, cela n'impacte pas les volumes car ils sont extérieurs au conteneur.

```bash
docker run -ti --rm -v $(pwd)/monFichier:/monFichier bash
```

- `-v cheminAbsoluSurHote:cheminAbsoluSurConteneur` permet de monter un volume sur le conteneur
- `$(pwd)` correspond au chemin courant
- en montant un volume, on récupère le contenu du fichier dans notre conteneur et les modifications faitent dans le conteneur seront conservées et donc accessibles en dehors du conteneur même quand celui-ci sera supprimé
- il faut faire attention au point de montage sur le conteneur pour ne pas occulter l'ancien contenu du dossier, car un volume est prioritaire. Cela peut rendre le conteneur inutilisable
- attention à ne pas monter dans le conteneur un dossier sensible de l'hôte. `/` donnerait une vue complète sur tout le système de fichiers de l'hôte
- `:ro` permet de monter un volume en lecture seule : `-v cheminAbsoluSurHote:cheminAbsoluSurConteneur:ro`

## Le réseau, partie 1

Nous allons voir comment configurer un conteneur pour communiquer avec lui via le réseau TCP/IP. Pour communiquer avec un conteneur, il faudra configurer la partie réseau au lancement d'un conteneur. Un conteneur peut ouvrir des ports de communication, mais par défaut, ça reste local au conteneur, ça veut dire que même si un port à été ouvert sur le conteneur, ce dernier ne peut pas communiquer avec son hôte et que l'hôte ne pas communiquer avec le conteneur en utilisant ce port. On peut établir une passerelle entre l'hôte et le conteneur en mappant un port de communication de l'hôte vers un port de communication du conteneur.

```bash
docker run --rm nginx
```

Par défaut, _nginx_ (serveur web) ouvre le port 80 local au conteneur mais celui-ci n'est pas accessible via la machine hôte car le port n'est pas mappé avec un port de l'hôte.

```bash
docker run --rm -p 80:80 nginx
```

Ici, on a fait le mapping du port avec `-p 80:80`, où le port de l'hôte est à gauche des deux points, et le port du conteneur à droite. Cette fois-ci, le conteneur est accessible sur le port 80 de la machine hôte sur le port 80.

Ce n'est pas obligatoire de mapper les mêmes numéros de ports des deux côtés. D'ailleurs, ce n'est pas toujours possible si le port sur l'hôte est déjà utilisé car un port ne peut être utilisé qu'une fois.

Il est possible de faire le mapping sans spécifier le port de l'hôte, dans ce cas c'est l'hôte qui va choisir quel port il met à disposition et chercher quel port a été choisi : `docker run --rm -p 80 nginx`. Avec la commande `docker ps` ou `docker inspect <nom-conteneur>`, on retrouve les ports utilisés.

Il est possible d'inspecter une image pour regarder si elle expose des ports et quels sont leur numéro : `docker inspect nginx` nous permet de savoir que nginx ouvre le port 80.


## Détacher, attacher

`-ti` permet de s'attacher à un conteneur dès son lancement quand on le crée. Généralement, on n'a pas besoin de s'y attacher à son lancement, comme dans le cas d'un serveur web, on le laisse vivre en autonomie.
Si on souhaite lancer un conteneur en mode détaché, il faut utilisé l'option `-d`, et on récupère aussitôt la main avec le conteneur qui tourne toujours.

```bash
docker run -d -p 80:80 nginx
```

Pour se rattacher au conteneur, il faut utiliser la commande `docker attach <id ou nom conteneur>`. En faisant `Ctrl+C`, le conteneur est arrêté. Pour se détacher d'un conteneur sans l'arrêter, il faut avoir ajouter un `-ti` au lancement même si on le lance en mode détaché :
```bash
# création d'un conteneur en mode détaché, on récupère donc immédiatement la main
docker run -ti -d -p 80:80 nginx
# attachement au conteneur avec son id
docker attach f4d
# il faut ensuite taper Ctrl+P puis Ctrl+Q pour se détacher sans arrêter le conteneur
```

`docker logs <id ou nom conteneur>` permet d'afficher les logs d'un conteneur puis rend la main. L'option `-f` (pour *follow*) permet de suivre les logs sans rendre la main et ainsi voir les nouvelles lignes dans le fichier.


## Créer une image Docker

Création d'une image Docker à partir d'une image PHP à laquelle on va ajouter une extension :

```bash
# récupération de l'image PHP puis on se retrouve dans le conteneur en ligne de commande PHP
docker run -ti php
# docker exec permet de rentrer dans le conteneur déjà actif avec un bash
docker exec -ti 78493 bash # 78493 est l'id du conteneur
# installation d'une extension à PHP
docker-php-ext-install soap
# erreur, il manque une bibliothèque, on va don l'installer
apt-get update
apt-get install libxml2-dev
# une fois la lib installée, réinstallation de soap
docker-php-ext-install soap
# récupération d'un interpréteur de commandes PHP
php -a
new SoapClient('URL')
```

Si on supprime le conteneur, tout est perdu. On va donc créer une image qui fait tout cela. C'est une *recette de cuisine* écrite dans un fichier nommé **Dockerfile** par convention.

```bash
FROM php:7.0.31-cli # un dockerfile commence toujours par FROM pour dire l'image de base que l'on va utiliser
RUN apt-get update # RUN permet de dire que l'on va exécuter une commande
RUN apt-get install -y libxml2-dev # -y permet de répondre automatiquement yes à toutes les questions
RUN docker-php-ext-install soap
```

Construction d'une image à partir d'un dockerfile :
- avec la commande `build`
- `-t` donne un tag (version) à l'image (nom:tag)
- chemin du dockerfile (. s'il est dans répertoire courant et qu'il s'appelle Dockerfile, sinon ajout de `-f nomDockerfile`)

```bash
# création de l'image
docker build -t php_soap:7.0.31 .
# vérification si l'image est bien dans le cash
docker images
# création d'un conteneur de notre image
docker run -ti php_soap:7.0.31
```

Pour mettre à jour notre image en changeant par exemple la version de l'image de base que l'on utilise, il faut :
- modifier le dockerfile `FROM php:7.2.8-cli`
- regénérer une nouvelle image : `docker build -t php_soap:7.2.8 .`


## Le réseau, partie 2

Docker offre différents pilotes (*drivers*) de réseaux :
- 5 pilotes natifs
- d'autres utilisables sous forme de plugin

Pour spécifier un type de réseau, il faut ajouter dans une commande Docker `--network <type>`.

Les pilotes natifs :
- `none` : absence total de réseau au conteneur
    - exemple : `docker run -ti --rm --network none bash`
    - le conteneur est isoé
- `bridge` : pilote par défaut quand rien n'est précisé
    - bon compromis pour l'isolation des conteneurs tout en offrant une connectivité aux conteneurs
    - si on lance 2 conteneurs avec la commande `docker run -ti --rm bash`, ils peuvent se voir et communiquer, ainsi que l'hôte peut aussi les voir car il est lui aussi sur le bridge par défaut
    - le pilote bridge peut êter **clôné**, c'est-à-dire créer d'autres bridges fonctionnant sur le même principe. Seul les conteneurs utilisant ce bridge clôné peuvent se voir
    - création d'un bridge clôné : `docker network create --driver=bridge mon_bridge`
    - créaton de 2 conteneurs en utilisant ce bridge : `docker run -ti --rm --network==mon_bridge bash`. Les 2 conteneurs peuvent toujours communiquer entre-eux car ils sont sur le même réseau mais l'hôte ne peut plus les voir car il n'est pas sur le même réseau
    - lorsque'un crée un réseau bridge, une plage d'adresse IP lui est effecté. Docker gère un DNS interne, uniquement pour les réseaux bridges clônés. Les conteneurs sont accessibles par leur adresse IP et donc aussi par leur nom
    - les bridges clônés peuvent être connectés/déconnectés dynamiquement des conteneurs déjà actifs. `docker network connect mon_bridge srv2` permet de rattacher le conteneur srv2 au bridge *mon_bridge*
- `host` : permet à un conteneur de partager la même réseau que l'hôte. En bridge, lorsqu'on ouvre un port sur un conteneur, il n'est pas ouvert sur l'hôte par défaut, il faut mapper un port du conteneur avec un port de l'hôte. Avec le pilote *host*, ouvrir un port sur le conteneur revient à ouvrir ce port sur l'hôte. Il n'y a plus d'isolation sur le réseau
    - pilote assez peu courant, qui peut servir dans certains cas
    - va un peu à l'encontre du principe d'isolation de Docker
- `overlay` permet de connecter entre-eux des conteneurs qui tournent sur des hôtes différents
- `macvlan` usage spécifique et peu courant

La commande **network** :
- `network create` permet de créer un pilote
- `network ls` permet de lister les réseaux
- `network rm` permet de supprimer un réseau (les réseaux natifs ne peuvent pas être supprimés)


TODO : vérifier qu'en bridge clôné, l'hôte ne peut plus voir les conteneurs car ils sont sur un bridge clôné alors que l'hôte est sur le bridge par défaut


## Les volumes, partie 2

Pour mapper un volume (fichier ou dossier) entre l'entre et le conteneur, on utilise la commande `-v cheminAbsoluSurHote:cheminAbsoluSurConteneur`. Cela masque ce qu'il y a sur le conteneur avec ce qu'il y a sur l'hôte en cas de conflit ed nom. On parle de **volumes mappés**.

Il existe aussi un autre type de volume : les **volumes managés**. Docker a la main dessus, alors que les volumes mappés sont entre les mains de l'OS. Les performances sont les mêmes.
- il faut d'abord créer le volume : `docker volume create mes_donnees`
- pour voir les volumes, il faut faire `docker volume ls`
- pour voir où sont stockés les volumes managés, il faut utiliser la commande `docker volume inspect mes_donnees`. On voit que les volumes sont dans l'espace de travail de docker (/var/lib/docker/volumes...)
- pour monter le volume managé, il faut renseigner le nom du volume managé au lieu du chemin absolu du volume mappé : `docker run --rm -ti -v mes_donnees:/src bash`
- concernant le masquage des fichiers et dossiers déjà présent dans le conteneur, le comportement est légèrement différent avec les volumes managés : lorsqu'on monte un **volume managé vide**, le dossier de montage du conteneur est transféré dans le volume managé. Le contenu du volume managé peut être lu depuis l'hôte ou le conteneur, de manière persistance au conteneur


## Docker Compose

Docker Compose est un **chef de cuisine** qui permet d'écrire des recettes qu'il va exécuter, et créer des environnements multi-conteneurs communiquant entre-eux (par exemple, une BDD et un serveur web).

Exemple avec Wordpress où il faut un conteneur **wordpress** + un conteneur **mysql**.

Docker Compose utilise un fichier de configuration qui se nomme **docker-compose.yml**

```yml
version: "3" # version de Docker Compose
services: # liste des conteneurs dont on a besoin
    db:
        image: mysql:5.7
        environnement: # il faut définir certains paramètres à la construction du conteneur
            - MYSQL_ROOT_PASSWORD=toto
            - MYSQL_DATEBASE=mabase
            - MYSQL_USER=gaetan
            - MYSQL_PASSWORD=varlet
        networks:
            - monreseaubridge
        volumes:
            - ./data/db:/var/lib/mysql
    wordpress:
        image: wordpress:4.9 # image source que l'on va utiliser
        ports:
            - 80:80 # liste des ports que l'on veut mapper
        environnement:
            - WORDPRESS_DB_HOST=db # nom du conteneur contenant mysql
            - WORDPRESS_DB_USER=gaetan
            - WORDPRESS_DB_PASSWORD=varlet
            - WORDPRESS_DB_NAME=mabase
        networks:
            - monreseaubridge
        volumes:
            - ./data/wp:var/www/html
networks: # configuration du réseau pour que nos conteneurs communiquent entre-eux
    monreseaubridge: # création d'un réseau bridgé clôné
```

Pour construire l'environnemnt, il faut exécuter la commande `docker-compose up`. Il ne nous rend pas la main et affiche les logs. `Ctrl+C` permet d'arrêter tous les conteneurs et de récupérer la main.
- pour lancer doker-compose en tâche de fond et récupérer la main, il va falloir mettre `-d`
- pour accéder aux logs, il faut faire `docker-compose logs -f`, `Ctrl+C` quitte les logs mais n'arrête pas l'environnement
- `docker-compose stop` permet d'arrêter l'environnement sans supprimer les conteneurs
- `docker compose start` permet de redémarrer les conteneurs
- `docker compose down` permet de supprimer un environnement. Tout est supprimé sauf les volumes. Ajouter `-v` permet de supprimer également les volumes

On peut refaire un fichier **docker-compose.yml** non plus avec des volumes mappés mais avec des **volumes managés** :
```yml
version: "3"
services:
    db:
        image: mysql:5.7
        environnement:
            - MYSQL_ROOT_PASSWORD=toto
            - MYSQL_DATEBASE=mabase
            - MYSQL_USER=gaetan
            - MYSQL_PASSWORD=varlet
        networks:
            - monreseaubridge
        volumes:
            - db:/var/lib/mysql # utilisation du volumes db déclaré en dessous
    wordpress:
        image: wordpress:4.9
        ports:
            - 80:80
        environnement:
            - WORDPRESS_DB_HOST=db
            - WORDPRESS_DB_USER=gaetan
            - WORDPRESS_DB_PASSWORD=varlet
            - WORDPRESS_DB_NAME=mabase
        networks:
            - monreseaubridge
        volumes:
            - wp:var/www/html # utilisation du volumes wp déclaré en dessous
volumes: # création d'une partie volumes pour faire des volumes managés
    db:
    wp:
networks:
    monreseaubridge:
```

## Portainer

Portainer est une interface web permettant de piloter graphiquement les conteneurs d'un ou plusieurs hôtes. Il est possible d'installer un image docker de portainer, et de faire des tâches d'administration via Portainer pour ne pas les faire en ligne de commande.
