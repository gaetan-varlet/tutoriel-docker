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
- `-ti` : `-t` dit à Docker qu'on souhaite avoir accès à un terminal  en pseudo TTY, c'est-à-dire avec l'entrée standard (clavier) et la sortie standard (écran). `-i` démarre le conteneur en mode interactif. On va interagir avec le conteneur au clavier via le pseudo TTY que l'on a créé.
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
