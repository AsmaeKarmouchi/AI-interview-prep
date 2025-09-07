# Docker & Kubernetes - Déploiement & DevOps Q/A

## Partie 1 : Docker

### Q1: Qu'est-ce que Docker et pourquoi l'utiliser ?
**Réponse :**
Docker permet de créer des conteneurs pour isoler et déployer des applications de façon portable et reproductible.

### Q2: Différence entre VM et conteneur ?
**Réponse :**
- VM : virtualise tout un OS, plus lourd.
- Conteneur : partage le noyau de l’OS hôte et n’isole que les processus, plus léger et rapide.

### Q2b : Quelle est la différence entre une image et un conteneur ?
**Réponse :**
- Une image est un modèle en lecture seule, une sorte de "plan" (template).
- Un conteneur est une instance en cours d’exécution de cette image.

### Q3: Quels sont les composants principaux d'un Dockerfile ?
**Réponse :**
- `FROM` : image de base
- `RUN` : exécute des commandes (installation de paquets, etc.)
- `COPY` / `ADD` : copie des fichiers dans l'image
- `CMD` / `ENTRYPOINT` : commande lancée au démarrage du conteneur
- `EXPOSE` : expose un port
- `ENV` : définit des variables d'environnement
- `WORKDIR` : définit le répertoire de travail

### Q4: Qu'est-ce qu'un volume dans Docker ?
**Réponse :**
Un volume permet de persister les données générées par un conteneur, même après son arrêt ou suppression.

### Commandes Docker courantes
- Créer et démarrer un nouveau conteneur à partir d’une image :
```bash
docker run <image>
```
- Démarrer un conteneur existant (arrêté) :
```bash
docker start <id>
```
- Exécuter une commande dans un conteneur déjà en cours d’exécution :
```bash
docker exec -it <id> <commande>
```
- Lister les conteneurs actifs :
```bash
docker ps
```
- Lister tous les conteneurs :
```bash
docker ps -a
```
- Lister les images locales :
```bash
docker images
```
- Supprimer un conteneur :
```bash
docker rm <id>
```
- Supprimer une image :
```bash
docker rmi <id>
```
- Créer un volume :
```bash
docker volume create <nom>
```
- Monter un volume :
```bash
docker run -v <volume>:/chemin/dans/conteneur <image>
```
- Monter un dossier de l’hôte (bind mount) :
```bash
docker run -v /chemin/host:/chemin/conteneur <image>
```

### Q5: Qu'est-ce qu'un multistage build dans Docker ?
**Réponse :**
Un multistage build permet de créer des images Docker optimisées en utilisant plusieurs étapes dans le même Dockerfile. On peut compiler dans une étape, puis copier uniquement les fichiers nécessaires dans l'image finale, ce qui réduit la taille et améliore la sécurité.

### Q6: À quoi sert l'option -p dans Docker ?
**Réponse :**
L'option `-p` (publish) permet de mapper un port du conteneur vers un port de la machine hôte.
```bash
docker run -p 8080:80 <image>
```
Expose le port 80 du conteneur sur le port 8080 de l'hôte.

### Qu’est-ce que Docker Compose ?
**Réponse :**
Outil permettant de définir et gérer des applications multi-conteneurs avec un fichier `docker-compose.yml`.

### Types de réseaux Docker
- `bridge` (par défaut) : communication via NAT
- `host` : partage le réseau de l’hôte
- `overlay` : pour les clusters Docker Swarm
- `macvlan` : chaque conteneur a sa propre adresse MAC

### Nettoyage Docker
- Supprimer les conteneurs arrêtés :
```bash
docker container prune
```
- Supprimer les réseaux inutilisés :
```bash
docker network prune
```
- Supprimer les images inutilisées :
```bash
docker image prune -a
```

### Limiter l’utilisation CPU et mémoire d’un conteneur
```bash
docker run --cpus=2 --memory=1g <image>
```

---

## Partie 2 : Kubernetes

### Q1: Qu'est-ce que Kubernetes ?
**Réponse :**
Kubernetes est un orchestrateur de conteneurs permettant de gérer le déploiement, la scalabilité et la résilience des applications.

### Q2: Expliquer le concept de pod dans Kubernetes.
**Réponse :**
Un pod est l'unité de base de déploiement dans Kubernetes, regroupant un ou plusieurs conteneurs partageant le même réseau et stockage.

### Q3: Qu'est-ce qu'un service dans Kubernetes ?
**Réponse :**
Un service expose un ou plusieurs pods sur le réseau, permettant la communication interne ou externe.

### Q3b: Quels sont les types de services dans Kubernetes ?
**Réponse :**
- `ClusterIP` : accès interne au cluster
- `NodePort` : expose le service sur un port de chaque nœud
- `LoadBalancer` : expose le service via un load balancer externe
- `ExternalName` : mappe le service à un nom DNS externe

### Q4: Qu'est-ce qu'un namespace dans Kubernetes ?
**Réponse :**
Un namespace permet d'isoler des ressources (pods, services, etc.) dans un cluster Kubernetes.

### Q5: Qu'est-ce qu'un ConfigMap et un Secret ?
**Réponse :**
- ConfigMap : stocke des données de configuration non sensibles
- Secret : stocke des informations sensibles (mots de passe, clés API)

### Q6: Qu'est-ce qu'un ingress ?
**Réponse :**
Un ingress gère l'accès HTTP/HTTPS externe vers les services du cluster, avec routage, SSL, etc.

### Q7: Comment scaler une application dans Kubernetes ?
**Réponse :**
En augmentant le nombre de réplicas d'un déploiement (horizontal scaling) ou en allouant plus de ressources (vertical scaling).

### Q8: Difference between Docker and Kubernetes?
**Réponse :**
- **Docker**: Containerization.
- **Kubernetes**: Orchestration of containers at scale.

### Q9: How to deploy an LLM in Kubernetes?
**Réponse :**
Build Docker image → push to registry → define K8s deployment (YAML) → expose via service/ingress.

### Q10: Qu'est-ce que le CI/CD et comment l'utiliser avec Docker/K8s ?
**Réponse :**
CI/CD automatise le build, les tests et le déploiement. On peut intégrer Docker pour la création d'images et Kubernetes pour le déploiement continu via des pipelines (GitHub Actions, GitLab CI, Jenkins).

---

## 📝 Cheat Sheet - Commandes Docker & Kubernetes

### Docker - Commandes rapides

| Action | Commande | Description |
|--------|----------|------------|
| Lister conteneurs actifs | `docker ps` | Conteneurs en cours d’exécution |
| Lister tous les conteneurs | `docker ps -a` | Même ceux arrêtés |
| Lister les images | `docker images` | Voir toutes les images locales |
| Démarrer un conteneur | `docker start <id>` | Démarre un conteneur existant |
| Créer et exécuter un conteneur | `docker run <image>` | Nouveau conteneur à partir d’une image |
| Exécuter une commande dans un conteneur | `docker exec -it <id> bash` | Accès interactif |
| Supprimer un conteneur | `docker rm <id>` | Supprime le conteneur |
| Supprimer une image | `docker rmi <id>` | Supprime l’image locale |
| Créer un volume | `docker volume create <nom>` | Stockage persistant |
| Nettoyer conteneurs arrêtés | `docker container prune` | Libérer de l’espace |
| Publier un port | `docker run -p <host>:<container> <image>` | Mapper port hôte → conteneur |

---

### Kubernetes (kubectl) - Commandes rapides

| Action | Commande | Description |
|--------|----------|------------|
| Lister les pods | `kubectl get pods` | Tous les pods dans le namespace courant |
| Lister les services | `kubectl get svc` | Affiche les services |
| Lister les déploiements | `kubectl get deployments` | Affiche les déploiements |
| Créer à partir d’un fichier YAML | `kubectl apply -f <fichier>.yaml` | Crée ou met à jour une ressource |
| Supprimer une ressource | `kubectl delete -f <fichier>.yaml` | Supprime la ressource |
| Accéder à un pod | `kubectl exec -it <pod> -- bash` | Accès shell au conteneur |
| Afficher logs d’un pod | `kubectl logs <pod>` | Voir les journaux |
| Scaler un déploiement | `kubectl scale deployment <nom> --replicas=<n>` | Modifier nombre de réplicas |
| Décrire un pod ou ressource | `kubectl describe pod <nom>` | Détails complets d’un pod |
| Changer de namespace | `kubectl config set-context --current --namespace=<nom>` | Travailler dans un autre namespace |
| Obtenir tous les objets dans un namespace | `kubectl get all` | Pods, services, déploiements, etc. |

---


