## Mock Interviews - Q/A Simulées

### Q1: Comment choisir le bon modèle pour une tâche NLP ?
**Réponse:**
Analyser la nature des données, la taille du corpus, la complexité de la tâche et les ressources disponibles. Tester plusieurs modèles et comparer leurs performances.

### Q2: Expliquez le processus de déploiement d'un modèle ML en production.
**Réponse:**
Préparer le modèle, le packager (Docker), créer une API (FastAPI, Flask), monitorer les performances, mettre en place la CI/CD et la gestion des versions.

### Q3: Quelles métriques utiliser pour évaluer un modèle de classification ?
**Réponse:**
Accuracy, précision, rappel, F1-score, courbe ROC-AUC.

### Q4: Comment gérer les biais dans les données ?
**Réponse:**
Analyser les sources de biais, équilibrer les classes, utiliser des techniques d'échantillonnage, valider sur des jeux de données diversifiés.


**Q: How do you reduce hallucinations in LLMs?**  
- Use RAG, grounding with external data, fine-tuning, better prompt engineering.

**Q: How would you deploy a GenAI model in production?**  
- Containerization (Docker), orchestration (K8s), monitoring, scaling with APIs, CI/CD.

**Q: Scenario**: You have latency issues in an LLM service. What steps do you take?  
- Profile inference pipeline.  
- Use quantization, distillation, or caching.  
- Optimize batch size, hardware acceleration (GPU/TPU).