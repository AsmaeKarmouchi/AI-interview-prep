
# Docker & Kubernetes - Déploiement & DevOps Q/A

## Partie 1 : Docker

### Q1: Qu'est-ce que Docker et pourquoi l'utiliser ?
**Réponse:**
Docker permet de créer des conteneurs pour isoler et déployer des applications de façon portable et reproductible.

### Q2: Différence entre VM et conteneur ?
**Réponse:**
VM : virtualise tout un OS, plus lourd. 
Conteneur : partage le noyau de l’OS hôte et n’isole que les processus, plus léger et rapide.
### Q2b : Quelle est la différence entre une image et un conteneur ?
**Réponse:**
- Une image est un modèle en lecture seule, une sorte de "plan" (template).
- Un conteneur est une instance en cours d’exécution de cette image.

### Q3: Quels sont les composants principaux d'un Dockerfile ?
**Réponse:**
- `FROM` : image de base
- `RUN` : exécute des commandes (installation de paquets, etc.)
- `COPY` / `ADD` : copie des fichiers dans l'image
- `CMD` / `ENTRYPOINT` : commande lancée au démarrage du conteneur
- `EXPOSE` : expose un port
- `ENV` : définit des variables d'environnement
- `WORKDIR` : définit le répertoire de travail
pour construire une image Docker automatiquement.

### Q4: Qu'est-ce qu'un volume dans Docker ?
**Réponse:**
Un volume permet de persister les données générées par un conteneur, même après son arrêt ou suppression.

### Commandes et pratique
 Quelle est la différence entre docker run, docker start et docker exec ?

- docker run → crée et démarre un nouveau conteneur à partir d’une image.
- docker start → démarre un conteneur existant (arrêté).
- docker exec → exécute une commande dans un conteneur déjà en cours d’exécution.
- Conteneurs actifs : docker ps
- Tous les conteneurs : docker ps -a
- Images locales : docker images
- Supprimer un conteneur : docker rm <id>
- Supprimer une image : docker rmi <id>
- Comment persister les données dans Docker ?
- - Volumes (docker volume create) → stockage géré par Docker.
- - Bind mounts → montage direct d’un dossier de l’hôte dans le conteneur.

### Q5: Qu'est-ce qu'un multistage build dans Docker ?
**Réponse:**
Un multistage build permet de créer des images Docker optimisées en utilisant plusieurs étapes dans le même Dockerfile. On peut compiler dans une étape, puis copier uniquement les fichiers nécessaires dans l'image finale, ce qui réduit la taille et améliore la sécurité.

### Q6: À quoi sert l'option -p dans Docker ?
**Réponse:**
L'option `-p` (publish) permet de mapper un port du conteneur vers un port de la machine hôte. Exemple : `docker run -p 8080:80` expose le port 80 du conteneur sur le port 8080 de l'hôte.

### Qu’est-ce que Docker Compose ?
 Outil permettant de définir et gérer des applications multi-conteneurs avec un fichier docker-compose.yml. On y définit les services, volumes, réseaux, etc.
 
### Quels sont les types de réseaux Docker ?
- bridge (par défaut) → conteneurs communiquent via NAT.
- host → conteneur partage le réseau de l’hôte.
- overlay → pour les clusters Docker Swarm.
- macvlan → chaque conteneur a sa propre adresse MAC.

### Vous êtes en charge de l’environnement Docker de l’entreprise et vous remarquez de nombreux conteneurs arrêtés et réseaux inutilisés qui prennent de la place. Que faites-vous ?
 Je nettoie l’environnement en utilisant :
- docker container prune pour supprimer les conteneurs arrêtés.
- docker network prune pour supprimer les réseaux inutilisés.
- docker image prune -a pour supprimer les images inutilisées.
Cela permet de libérer de l’espace disque et de garder un environnement sain.

### Comment gérez-vous le stockage persistant dans Docker ?
- Volumes (solution recommandée, gérés par Docker).
- Bind mounts (montage d’un répertoire de l’hôte).
Cela permet de conserver les données même si le conteneur est supprimé.

### Une entreprise veut créer des milliers de conteneurs. Existe-t-il une limite au nombre de conteneurs que Docker peut exécuter ?
Docker n’impose pas de limite stricte. Le vrai facteur limitant est la capacité matérielle (CPU, RAM, I/O). Plus le serveur est puissant, plus on peut exécuter de conteneurs.

### Comment limiter l’utilisation CPU et mémoire d’un conteneur ?

docker run --cpus=2 --memory=1g mycontainer
Cela limite le conteneur à 2 CPU et 1 Go de RAM.

---

## Partie 2 : Kubernetes

### Q1: Qu'est-ce que Kubernetes ?
**Réponse:**
Kubernetes est un orchestrateur de conteneurs permettant de gérer le déploiement, la scalabilité et la résilience des applications.

### Q2: Expliquer le concept de pod dans Kubernetes.
**Réponse:**
Un pod est l'unité de base de déploiement dans Kubernetes, regroupant un ou plusieurs conteneurs partageant le même réseau et stockage.


### Q3b: Quels sont les types de services dans Kubernetes ?
**Réponse:**
- `ClusterIP` : accès interne au cluster
- `NodePort` : expose le service sur un port de chaque nœud
- `LoadBalancer` : expose le service via un load balancer externe
- `ExternalName` : mappe le service à un nom DNS externe

### Q4: Qu'est-ce qu'un namespace dans Kubernetes ?
**Réponse:**
Un namespace permet d'isoler des ressources (pods, services, etc.) dans un cluster Kubernetes.

### Q5: Qu'est-ce qu'un ConfigMap et un Secret ?
**Réponse:**
ConfigMap stocke des données de configuration non sensibles, Secret stocke des informations sensibles (mots de passe, clés API).

### Q6: Qu'est-ce qu'un ingress ?
**Réponse:**
Un ingress gère l'accès HTTP/HTTPS externe vers les services du cluster, avec routage, SSL, etc.

### Q7: Comment scaler une application dans Kubernetes ?
**Réponse:**
En augmentant le nombre de réplicas d'un déploiement (horizontal scaling) ou en allouant plus de ressources (vertical scaling).

### Q8: Difference between Docker and Kubernetes?
**Réponse:**
- **Docker**: Containerization.
- **Kubernetes**: Orchestration of containers at scale.

### Q9: How to deploy an LLM in Kubernetes?
**Réponse:**
Build Docker image → push to registry → define K8s deployment (YAML) → expose via service/ingress.

### Q10: Qu'est-ce que le CI/CD et comment l'utiliser avec Docker/K8s ?
**Réponse:**
CI/CD automatise le build, les tests et le déploiement. On peut intégrer Docker pour la création d'images et Kubernetes pour le déploiement continu via des pipelines (GitHub Actions, GitLab CI, Jenkins).
