# TP Docker

## Introduction

Ce TP a pour but de vous familiariser avec Docker et Docker Compose.
Dans un premier temps, nous allons _dockeriser_ une application Java qui utilise OpenCV. Cela consiste à créer un Dockerfile contenant les dépendances nécessaires pour compiler et exécuter l'application.
Ensuite, nous allons configurer Docker Compose pour déployer plusieurs instances de l'application avec un serveur web en reverse proxy.

## Liens Utiles

<details>
<summary>Cliquer pour déplier</summary>

- [Documentation Docker](https://docs.docker.com/)
- [Tutoriel Docker](https://docs.docker.com/get-started/)
- [Reverse Proxy Nginx Automatisé pour Docker](http://jasonwilder.com/blog/2014/03/25/automated-nginx-reverse-proxy-for-docker/)
- [Compilation OpenCV sur Ubuntu](https://advancedweb.hu/2016/03/01/opencv_ubuntu/)

</details>

# Modalités de rendu

Tous les rendus TLC se font à travers le gitlab de l'ISTIC : https://gitlab2.istic.univ-rennes1.fr/

Si vous ne l'avez pas déjà fait, créez sur Gitlab ISTIC un groupe nommé `TLC_2025_<votre_nom>_<votre_prenom>` 

Pour chaque TP, vous devrez créer un projet dans ce groupe, nommé `TP<numero>_<votre_nom>_<votre_prenom>` et le projet finale nommé `Projet_<votre_nom>_<votre_prenom>`

Pour chaque TP, vous devrez ajouter votre enseignant en tant que membre du projet avec le rôle de "Reporter" pour permettre la correction.

### Étape 0: Prérequis
Il est fortement conseillé de dérouler ce TP sur une machine Linux (Ubuntu, Fedora, etc.) ou en utilisant une machine virtuelle (VirtualBox, Vagrant, etc.).

1. Installez Docker dans votre environnement de développement.
2. Clonez ce dépot.

## [PARTIE 1] Dockeriser une application

### Commandes Utiles

<details>
<summary>pour interagir avec les images</summary>

- `docker build -t <nom_image> .` : Construit une image Docker à partir d'un Dockerfile situé dans le répertoire courant et lui donne un nom.
- `docker rmi <nom_image>` : Supprime une image.
- `docker images` : Liste les images.
- `docker pull <nom_image>` : Télécharge une image depuis le Docker Hub.

</details>

<details>
<summary>pour interagir avec les conteneurs</summary>

- `docker run [nom_image]` : Démarre un conteneur à partir d'une image.
- `docker ps` : Liste les conteneurs en cours d'exécution.
- `docker exec -it [nom_conteneur] /bin/bash` : Exécute la commande `/bin/bash` dans un conteneur avec le mode interactif -] ça donne un shell (si bash est installé).
- `docker stop [nom_conteneur]` : Arrête un conteneur.
- `docker rm [nom_conteneur]` : Supprime un conteneur.
- `docker logs [nom_conteneur]` : Affiche les logs d'un conteneur.
</details>

D'autres commandes sont disponibles [ici](https://docs.docker.com/reference/cli/docker/).

### Tâche 1 : Créer une Image Docker à partir de `scratch`

> Créez un `Dockerfile` à partir d'une image vierge :
>    - Compilez le fichier `hello.c` (dans le dossier **step1.1**) avec `gcc -o hello_dyn hello.c`
>    - Créez un Dockerfile et utilisez `scratch` comme image de base.
>    - Ajoutez le fichier binaire `hello_dyn` et définissez la commande de démarrage.
>    - Build et exécutez l'image. Est-ce que le conteneur démarre correctement ?

> Compilez maintenant le fichier `hello.c` avec `gcc -static -o hello hello.c` et utilisez ce binaire au lieu de `hello_dyn`.
>   - Quelle est la différence entre les deux commandes de compilation ?

### Tâche 2 : Containériser une application existante

> Maintenant, on va utiliser comme image de base quelque chose de plus traditionnel, par exemple `ubuntu:18.04` (ou voir d'autres images pertinentes dans le Docker Hub).
>    - Installez les dépendances nécessaires pour OpenCV.
>    - Ajoutez les fichiers sources (dans le dossier **step1.2**) et compilez l'application.
>    - Assurez-vous d'avoir la bonne commande de démarrage du conteneur.

<details>
<summary>Cliquer pour des liens utiles</summary>

- [Dockerhub - repo images](https://hub.docker.com/)
- [Dockerfile Reference](https://docs.docker.com/reference/dockerfile)
</details>

<details>
<summary>Cliquer pour des indices</summary>
    
- Installez des dépendances comme OpenJDK, Maven, et OpenCV. 
```shell
    apt-get update
    apt-get install -y openjdk-8-jdk
    apt-get install -y maven
    apt-get install -f libpng16-16
    apt-get install -f libjasper1
    apt-get install -f libdc1394-22
```
- Installez OpenCV avec `mvn install:install-file -Dfile=./lib/opencv-3410.jar -DgroupId=org.opencv  -DartifactId=opencv -Dversion=3.4.10 -Dpackaging=jar` (Adaptez les path en fonction de votre Dockerfile).
- Utilisez `mvn package` pour compiler l'application.
- Utilisez `java -Djava.library.path=lib/ -jar target/fatjar-0.0.1-SNAPSHOT.jar` pour exécuter l'application. (Utilisez lib/ubuntuupperthan18 si vous avez comme image une version d'Ubuntu supérieure à 18.04)
- L'application est accessible sur le port 8080. Assurez-vous d'exposer ce port ou de le bind à un port de votre choix au démarrage du conteneur. Si tout est correct, http://localhost:8080 devrait être ouvert depuis votre navigateur.
</details>

### Tâche 3 : Améliorer le Dockerfile pour une image plus _light_

>Maintenant que vous avez une image fonctionnelle, vous allez essayer de la rendre plus légère. 
>
>Proposez un nouveau fichier Dockerfile qui permet de créer une image de taille réduite.

<details>
<summary>Cliquer pour des liens utiles</summary>

- [Build Multi-Stage Docker 1](https://learnk8s.io/blog/smaller-docker-images)
- [Build Multi-Stage Docker 2](https://docs.docker.com/develop/develop-images/multistage-build/)

</details>

## [PARTIE 2] Configurer un reverse proxy sous Docker
### Tâche 1 : Simple reverse proxy avec ligne de commande `docker`

<details>
<summary> Cliquer pour des liens utiles</summary>

Pour le nginx en reverse proxy, nous allons partir de l'image [suivante](https://github.com/jwilder/nginx-proxy).

L'explication du fonctionnement est disponible [ici](http://jasonwilder.com/blog/2014/03/25/automated-nginx-reverse-proxy-for-docker/).
</details>

> Si vous n'avez pas la tête à lire ça, la version abrégée est que le reverse proxy vous permet tout un tas de choses, y compris de gérer le fait que les containers ont des adresses IP (un peu) trop dynamiques, ce qui fait qu'à chaque changement/lancement de container, il y aurait des problèmes de binding de port. Le reverse proxy va vous permettre de cacher ces aspects-là, puisqu'ils seront gérés par ce composant. Ainsi, les chargements de versions modifiées de votre service n'auront pas besoin d'une gestion fine à la main des connexions, les différents utilisateurs qui voudront envoyer des requêtes simultanées au même service ne seront pas embêtés par des ports qui ne sont pas accessibles, etc.

- Lancement de nginx en reverse proxy :
> 
> ```bash
> docker run -d -p 8080:80 -v /var/run/docker.sock:/tmp/docker.sock -t jwilder/nginx-proxy
> ```
> 
> ⚠️ Pour certaines installations comme sur la dernière édition de Fedora, les règles de sécurité par défaut ont évolué. Pour que le container puisse accéder à la socket Docker, il faut ajouter l'option suivante :
> 
> ```bash
> docker run --security-opt=label:type:docker_t -d -p 8080:80 -v /var/run/docker.sock:/tmp/docker.sock -t jwilder/nginx-proxy
> ```
> 
- Si vous êtes sur votre propre portable, modifiez votre fichier `/etc/hosts` pour faire correspondre **m** vers localhost. Ce serait à faire sur votre gestionnaire de nom de domaine en temps normal.
> Vous devez avoir une ligne qui ressemble à cela :
> 
> ```txt
> 127.0.0.1    localhost localhost.localdomain localhost4 localhost4.localdomain m
> ```
> 
> Pour ceux qui n'ont pas les droits root, exécutez les commandes suivantes :
> 
> ```bash
> echo 'm localhost' >> ~/.hosts
> export HOSTALIASES=~/.hosts
> curl m:8080
> ```
> 
-  Puis créez plusieurs fenêtres dans votre terminal. Ce seront vos différentes machines host émulées. Vous pouvez en créer au moins 3 ou 4. Dans ces terminaux, lancez la commande suivante pour tester votre reverse proxy :
> 
> ```bash
> docker run -e VIRTUAL_HOST=m -t -i nginx
> ```
> 
- Testez votre reverse proxy en lançant la commande suivante dans votre terminal originel :
> 
> ```bash
> curl m:8080
> ```
> 
> En l'exécutant plusieurs fois et suffisamment rapidement, vous devriez voir tantôt une fenêtre terminator se mettre à jour, tantôt une autre. C'est l'effet du load balancer (un autre service qui est géré par votre nginx).
> 
> En tapant la commande suivante, vous pouvez regarder le fichier de configuration nginx qui sera généré à l'adresse suivante `/etc/nginx/conf.d/default.conf`. 
> 
> (N'oubliez pas de remplacer `865c1e67a00e` par l'id de votre nginx en reverse proxy (`docker ps`) pour récupérer la liste des containers en cours d'exécution) :
> ```bash
> docker exec -it 865c1e67a00e bash
> ```
> 

> ️️⚠️ N'oubliez pas de tuer les conteneurs lancés pour libérer des ressources :
> 
> ```bash
> docker ps # pour avoir la liste
> docker kill "IDDOCKER" # pour tuer un docker
> ```


### Tâche 2 : Configurer le reverse proxy dans un fichier Docker Compose
<details>
<summary>pour interagir avec un deploiement compose</summary>

- `docker-compose up` : Démarre les services.
- `docker-compose down` : Arrête les services.
- `docker-compose up -f <fichier>` : Démarre les services à partir d'un fichier spécifique.
- D'autres commandes sont disponibles [ici](https://docs.docker.com/reference/cli/docker/compose/).
</details>

> - Créez un fichier **docker-compose.yml** avec `jwilder/nginx-proxy`

<details>
<summary>Cliquer pour un exemple</summary>

> Presque toutes les commandes Docker peuvent être traduites en fichier **docker-compose.yml**. Cela permet de "scripter" le lancement de plusieurs conteneurs et surtout permet de simplifier la communication entre eux. 
> 
> Par exemple, le fichier compose suivant permet de lancer deux conteneurs:
> ```yaml
> version: '3'
> services:
>  serviceA:
>   image: debian
>   command: ping serviceB
>  serviceB:
>   image: debian
>   command: sleep 1000
> ```
> Le serviceA peut simplement ping le serviceB en utilisant son nom de service.

> Pour le reverse proxy, le fichier **docker-compose.yml** pour démarrer pourrait ressembler à ceci:
>```yaml
> version: '3'
> services:
>      nginx-proxy:
>        image: jwilder/nginx-proxy
>        ports:
>          - "8080:80"
>        volumes:
>          - /var/run/docker.sock:/tmp/docker.sock
>```
</details>


> - Ajoutez un service nginx classique qui utiliserait le reverse proxy et donnez lui un nom vhost.
> - Assurez-vous que votre fichier **/etc/hosts** contient une entrée pour le nom de domaine que vous avez choisi (vhost).
> - Vérifiez que tout fonctionne correctement en accédant à l'URL du vhost.


<details>
<summary>Cliquer pour des liens utiles</summary>

- [Docker compose services options](https://docs.docker.com/reference/compose-file/services/)
- [Repo officiel jwilder/nginx-proxy](https://hub.docker.com/r/jwilder/nginx-proxy/)
</details>

### Tâche 3  : Docker Compose avec 4 Instances
> Maintenant que vous avez familiarisé avec Docker Compose et le reverse proxy, ajoutez au **docker-compose.yaml** votre application Java en veillant à bien configurer le service.
> - Vérifiez que l'application fonctionne correctement en accédant à l'URL du vhost.
> - Modifiez le fichier compose pour permettre l'exécution de 4 instances de l'application.

<details>
<summary>Cliquer pour des liens utiles</summary>

- [Docker compose services options](https://docs.docker.com/reference/compose-file/services/)
</details>



## Rendu TP Docker
- Un fichier `Dockerfile` pour l'application Java.
- Un fichier `Dockerfile` pour l'application Java version light.
- Un fichier `docker-compose.yml` avec le reverse proxy, un service web simple et 4 instances de l'application Java.

# Annexes
<details>
<summary>Annexe 1 : Description de l'Application Java</summary>

# How to compile this application

Simple example of using OpenCV in a Web application build using jersey.

This application takes a picture using web browsers camera API (available  in modern browsers) 
and runs OpenCV face recognition algorithm (using [CascadeClassifier](http://docs.opencv.org/java/org/opencv/objdetect/CascadeClassifier.html) ) for it. If a face is detected a "troll face" is added  on top of it.

![Screenshot](/screenshot.png?raw=true "Screenshot")

This application was inspired by the ingenious ["Trollator" mobile Android application](https://play.google.com/store/apps/details?id=com.fredagapps.android.trollator).

1. OpenCV Installation for local Maven repository
---
OpenCV is a native library with Java bindings so you need to install this to your system.
 - *libopencv_java3410.so* installed in you java.library.path (
 - *opencv-3410.jar* availble for application

There are good instructions how to build OpenCV with Java bindings for your own platform here: http://docs.opencv.org/doc/tutorials/introduction/desktop_java/java_dev_intro.html

Once you have built the Java library you can install the resulting jar file to your local Maven repository using
```shell     
mvn install:install-file -Dfile=./lib/opencv-3410.jar -DgroupId=org.opencv  -DartifactId=opencv -Dversion=3.4.10 -Dpackaging=jar
```

2. Building this application
----
Once OpenCV jar library is available as a local Maven dependency, you can clone and build this application simply using Git and Maven:

```bash
     mvn install
```

And run the application using the embedded Jetty plugin in http://localhost:8080

```bash
     mvn package

     java -Djava.library.path=lib/ -jar target/fatjar-0.0.1-SNAPSHOT.jar
	# Do not forget to update the path to your opencv install in Main.java
	# You can change the image trollface ;)
```

</details>
