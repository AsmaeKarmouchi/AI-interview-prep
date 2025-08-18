## Git, GitHub, GitLab, CI/CD - Q/A

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


