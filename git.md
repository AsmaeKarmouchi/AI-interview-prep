git clone <url_du_depot>
git status
git add <nom_du_fichier>
git commit -m "message"
git log
git branch <nom_branche>
git checkout <nom_branche>
git checkout -b <nom_branche>
ou git switch -c <nom_branche>
git remote add origin <url_du_depot>
git push origin <nom_branche>
git fetch (récupère sans fusionner)
git pull (récupère + fusionne)
git checkout -- <fichier>
ou git restore <fichier>
git reset <fichier>
git checkout <id_commit>
git reset --hard <id_commit>
git stash apply
git stash pop
git stash drop stash@{n}
git diff
git diff --cached
git show <id_commit>
git rm --cached <fichier>
git rm <fichier>
git reset --hard HEAD~1

# Git, GitHub, GitLab, CI/CD - Q/A

## Les bases

- Initialiser un dépôt Git
```bash
git init
```
- Cloner un dépôt distant
```bash
git clone <url_du_depot>
```
- Afficher l’état des fichiers (modifiés, suivis, non suivis)
```bash
git status
```
- Ajouter un fichier à l’index (staging area)
```bash
git add <nom_du_fichier>
git add .   # tout ajouter
```
- Enregistrer les modifications avec un message
```bash
git commit -m "message"
```
- Afficher l’historique des commits
```bash
git log
```

## Branches

- Créer une nouvelle branche
```bash
git branch <nom_branche>
```
- Se déplacer sur une branche
```bash
git checkout <nom_branche>
git switch <nom_branche>   # nouvelle syntaxe
```
- Créer et se déplacer sur une branche en une seule commande
```bash
git checkout -b <nom_branche>
git switch -c <nom_branche>
```
- Fusionner une branche dans la branche courante
```bash
git merge <nom_branche>
```

## Travail avec un dépôt distant (GitHub, GitLab…)

- Lier un dépôt local à un dépôt distant
```bash
git remote add origin <url_du_depot>
```
- Envoyer (pousser) les modifications locales vers le dépôt distant
```bash
git push origin <nom_branche>
git push -u origin <nom_branche>   # première fois (lien branche-distante)
```
- Récupérer les mises à jour du dépôt distant
```bash
git fetch   # récupère sans fusionner
git pull    # récupère + fusionne
```

## Gestion des erreurs courantes

- Annuler les modifications non indexées (avant git add)
```bash
git checkout -- <fichier>
git restore <fichier>
```
- Retirer un fichier de l’index (après git add) mais garder les modifs locales
```bash
git reset <fichier>
git restore --staged <fichier>
```
- Revenir à un commit précédent sans perdre les changements
```bash
git checkout <id_commit>
```
- Réinitialiser complètement l’historique jusqu’à un commit donné (dangereux)
```bash
git reset --hard <id_commit>
```

## git stash

- Sauvegarder temporairement les modifications non commit ("mettre dans un tiroir")
```bash
git stash save "message"
```
- Lister les stashes
```bash
git stash list
```
- Restaurer le dernier stash sans le supprimer de la liste
```bash
git stash apply
```
- Restaurer ET supprimer le stash de la liste
```bash
git stash pop
```
- Supprimer un stash spécifique
```bash
git stash drop stash@{n}
```

## Rebase et historique

- Différence entre merge et rebase ?
  - `merge` → fusionne deux branches en conservant l’historique.
  - `rebase` → réapplique les commits d’une branche par-dessus une autre, pour un historique linéaire.
- Lancer un rebase sur main
```bash
git rebase main
```
- Réécrire les derniers commits (modifier message, fusionner commits)
```bash
git rebase -i HEAD~3   # pour les 3 derniers commits
```

## Diff et inspection

- Voir les différences entre fichiers modifiés et le dernier commit
```bash
git diff
```
- Voir les différences entre index (staging) et dernier commit
```bash
git diff --cached
```
- Voir le détail d’un commit précis
```bash
git show <id_commit>
```

## Nettoyage & corrections

- Supprimer un fichier du dépôt mais le garder en local
```bash
git rm --cached <fichier>
```
- Supprimer un fichier du dépôt et du disque
```bash
git rm <fichier>
```
- Annuler le dernier commit mais garder les fichiers modifiés
```bash
git reset --soft HEAD~1
```
- Annuler le dernier commit et supprimer aussi les modifs
```bash
git reset --hard HEAD~1
```





### Q1: Quelle est la différence entre Git et GitHub ?
**Réponse:**
Git est un système de gestion de versions décentralisé. GitHub est une plateforme en ligne pour héberger des dépôts Git et collaborer.

### Q2: Qu'est-ce qu'un merge conflict et comment le résoudre ?
**Réponse:**
Un merge conflict survient lorsque deux branches modifient la même partie d'un fichier. Il faut éditer le fichier, choisir les modifications à conserver, puis valider le merge.


**Q: What is `git commit`?**  
- Command to save staged changes into the local repository history.

**Q: What is `git stash`?**  
- Temporarily saves uncommitted changes so you can switch branches safely.

**Q: What is `git reflog`?**  
- Shows a log of all Git reference updates (useful for recovering lost commits).

**Q: What is `git reset`?**  
- Moves the current branch pointer to a different commit. Options:  
  - `--soft`: keeps changes staged.  
  - `--mixed`: keeps changes unstaged.  
  - `--hard`: discards changes.

**Q: What is `git merge`?**  
- Combines changes from one branch into another, creating a new merge commit if needed.



### Q3: Expliquez le principe de CI/CD.
**Réponse:**
CI/CD (Intégration Continue/Déploiement Continu) automatise les tests et le déploiement du code pour accélérer la livraison et améliorer la qualité.

### Q4: Citez des alternatives à GitHub.
**Réponse:**
GitLab, Bitbucket, Azure DevOps.

**Q: Difference between Git and GitHub/GitLab?**  
- **Git**: Version control system.  
- **GitHub/GitLab**: Platforms for hosting repositories, collaboration, CI/CD.

**Q: What is a Pull Request (PR) / Merge Request (MR)?**  
- Code contribution review process before merging into main branch.

**Q: What is CI/CD?**  
- **CI**: Continuous Integration (merge code + tests).  
- **CD**: Continuous Delivery/Deployment (automated release pipeline).





