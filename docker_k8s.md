
# Docker & Kubernetes - Déploiement & DevOps Q/A

## Partie 1 : Docker

### Q1: Qu'est-ce que Docker et pourquoi l'utiliser ?
**Réponse:**
Docker permet de créer des conteneurs pour isoler et déployer des applications de façon portable et reproductible.

### Q2: Différence entre VM et conteneur ?
**Réponse:**
VM : virtualise tout un OS, plus lourd. Conteneur : partage le noyau, plus léger et rapide.


### Q3b: Quels sont les composants principaux d'un Dockerfile ?
**Réponse:**
- `FROM` : image de base
- `RUN` : exécute des commandes (installation de paquets, etc.)
- `COPY` / `ADD` : copie des fichiers dans l'image
- `CMD` / `ENTRYPOINT` : commande lancée au démarrage du conteneur
- `EXPOSE` : expose un port
- `ENV` : définit des variables d'environnement
- `WORKDIR` : définit le répertoire de travail

### Q4: Qu'est-ce qu'un volume dans Docker ?
**Réponse:**
Un volume permet de persister les données générées par un conteneur, même après son arrêt ou suppression.

### Q5: Qu'est-ce qu'un multistage build dans Docker ?
**Réponse:**
Un multistage build permet de créer des images Docker optimisées en utilisant plusieurs étapes dans le même Dockerfile. On peut compiler dans une étape, puis copier uniquement les fichiers nécessaires dans l'image finale, ce qui réduit la taille et améliore la sécurité.

### Q6: À quoi sert l'option -p dans Docker ?
**Réponse:**
L'option `-p` (publish) permet de mapper un port du conteneur vers un port de la machine hôte. Exemple : `docker run -p 8080:80` expose le port 80 du conteneur sur le port 8080 de l'hôte.

### Q7: Why Docker in ML/AI?
**Réponse:**
Ensures reproducibility, consistency, easy deployment of models.

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