# Guide Complet - Entretien Technique SQL

## 1. Questions sur les Fondamentaux

### Q1: Différence entre INNER JOIN, LEFT JOIN, RIGHT JOIN, FULL OUTER JOIN

```sql
-- Tables d'exemple
CREATE TABLE employes (id INT, nom VARCHAR(50), dept_id INT);
CREATE TABLE departements (id INT, nom_dept VARCHAR(50));

-- INNER JOIN (intersection)
SELECT e.nom, d.nom_dept 
FROM employes e 
INNER JOIN departements d ON e.dept_id = d.id;

-- LEFT JOIN (tous les employés, même sans département)
SELECT e.nom, d.nom_dept 
FROM employes e 
LEFT JOIN departements d ON e.dept_id = d.id;

-- RIGHT JOIN (tous les départements, même sans employés)
SELECT e.nom, d.nom_dept 
FROM employes e 
RIGHT JOIN departements d ON e.dept_id = d.id;

-- FULL OUTER JOIN (union complète)
SELECT e.nom, d.nom_dept 
FROM employes e 
FULL OUTER JOIN departements d ON e.dept_id = d.id;
```

### Q2: Différence entre WHERE et HAVING

```sql
-- WHERE : filtre avant regroupement
SELECT dept_id, COUNT(*) 
FROM employes 
WHERE salaire > 50000 
GROUP BY dept_id;

-- HAVING : filtre après regroupement
SELECT dept_id, COUNT(*) 
FROM employes 
GROUP BY dept_id 
HAVING COUNT(*) > 5;
```
-- Règle Fondamentale
Quand GROUP BY est-il obligatoire ?
GROUP BY est obligatoire quand vous mélangez des colonnes normales avec des fonctions d'agrégation dans le SELECT.
```sql
SELECT COUNT(*) FROM employes;

SELECT dept_id, COUNT(*) FROM employes GROUP BY dept_id;

SELECT dept_id, COUNT(*) as nb_employes FROM employes GROUP BY dept_id;
```

Règle Détaillée : Toute colonne non-agrégée doit être dans GROUP BY
### Q3: Ordre d'exécution des clauses SQL

1. FROM (tables)
2. WHERE (filtrage des lignes)
3. GROUP BY (regroupement)
4. HAVING (filtrage des groupes)
5. SELECT (projection)
6. ORDER BY (tri)
7. LIMIT/TOP (limitation)

## 2. Requêtes Avancées

### Q4: Trouver le Nème salaire le plus élevé

```sql
-- Méthode 1: DENSE_RANK()
SELECT DISTINCT salaire 
FROM (
    SELECT salaire, DENSE_RANK() OVER (ORDER BY salaire DESC) as rang
    FROM employes
) ranked 
WHERE rang = 3; -- 3ème plus haut salaire

-- Méthode 2: LIMIT/OFFSET
SELECT DISTINCT salaire 
FROM employes 
ORDER BY salaire DESC 
LIMIT 1 OFFSET 2; -- 3ème plus haut

-- Méthode 3: Sous-requête
SELECT MAX(salaire) 
FROM employes 
WHERE salaire < (
    SELECT MAX(salaire) 
    FROM employes 
    WHERE salaire < (SELECT MAX(salaire) FROM employes)
);
```

### Q5: Employés avec salaire supérieur à la moyenne de leur département

```sql
SELECT e.nom, e.salaire, e.dept_id
FROM employes e
WHERE e.salaire > (
    SELECT AVG(e2.salaire)
    FROM employes e2
    WHERE e2.dept_id = e.dept_id
);
```

### Q6: Doublons et déduplication

```sql
-- Trouver les doublons
SELECT email, COUNT(*)
FROM utilisateurs
GROUP BY email
HAVING COUNT(*) > 1;

-- Supprimer les doublons (garder le plus récent)
DELETE u1 FROM utilisateurs u1
INNER JOIN utilisateurs u2
WHERE u1.id < u2.id AND u1.email = u2.email;

-- Avec ROW_NUMBER()
DELETE FROM utilisateurs
WHERE id NOT IN (
    SELECT min_id FROM (
        SELECT MIN(id) as min_id
        FROM utilisateurs
        GROUP BY email
    ) as temp
);
```

## 3. Fonctions de Fenêtrage (Window Functions)

### Q7: Calculs avec ROW_NUMBER, RANK, DENSE_RANK

```sql
SELECT 
    nom,
    salaire,
    ROW_NUMBER() OVER (ORDER BY salaire DESC) as position,
    RANK() OVER (ORDER BY salaire DESC) as rang,
    DENSE_RANK() OVER (ORDER BY salaire DESC) as rang_dense
FROM employes;

-- Par département
SELECT 
    nom,
    dept_id,
    salaire,
    ROW_NUMBER() OVER (PARTITION BY dept_id ORDER BY salaire DESC) as rang_dept
FROM employes;
```

### Q8: Calculs cumulatifs et moyennes mobiles

```sql
SELECT 
    date_vente,
    montant,
    SUM(montant) OVER (ORDER BY date_vente) as cumul,
    AVG(montant) OVER (ORDER BY date_vente ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) as moyenne_mobile_3
FROM ventes;
```

### Q9: LAG et LEAD pour comparaisons

```sql
SELECT 
    date_vente,
    montant,
    LAG(montant, 1) OVER (ORDER BY date_vente) as vente_precedente,
    montant - LAG(montant, 1) OVER (ORDER BY date_vente) as difference
FROM ventes;
```

## 4. Requêtes Complexes Courantes

### Q10: Top N par catégorie

```sql
-- Top 3 employés les mieux payés par département
SELECT * FROM (
    SELECT 
        nom, 
        dept_id, 
        salaire,
        ROW_NUMBER() OVER (PARTITION BY dept_id ORDER BY salaire DESC) as rn
    FROM employes
) ranked
WHERE rn <= 3;
```

### Q11: Analyse des ventes par période

```sql
SELECT 
    EXTRACT(YEAR FROM date_vente) as annee,
    EXTRACT(MONTH FROM date_vente) as mois,
    COUNT(*) as nb_ventes,
    SUM(montant) as total_ventes,
    AVG(montant) as vente_moyenne
FROM ventes
GROUP BY EXTRACT(YEAR FROM date_vente), EXTRACT(MONTH FROM date_vente)
ORDER BY annee, mois;
```

### Q12: Clients qui n'ont pas commandé depuis X mois

```sql
SELECT c.id, c.nom
FROM clients c
LEFT JOIN commandes co ON c.id = co.client_id 
    AND co.date_commande >= CURRENT_DATE - INTERVAL '6 months'
WHERE co.client_id IS NULL;
```

## 5. Optimisation et Performance

### Q13: Utilisation d'index

```sql
-- Créer des index
CREATE INDEX idx_employes_dept ON employes(dept_id);
CREATE INDEX idx_ventes_date ON ventes(date_vente);

-- Index composé
CREATE INDEX idx_employes_dept_salaire ON employes(dept_id, salaire);

-- Index partiel
CREATE INDEX idx_employes_actifs ON employes(id) WHERE statut = 'ACTIF';
```

### Q14: EXPLAIN PLAN

```sql
-- Analyser le plan d'exécution
EXPLAIN ANALYZE
SELECT e.nom, d.nom_dept
FROM employes e
JOIN departements d ON e.dept_id = d.id
WHERE e.salaire > 50000;
```

## 6. Questions Pièges Fréquentes

### Q15: NULL et comparaisons

```sql
-- ATTENTION: NULL != NULL
SELECT * FROM employes WHERE manager_id = NULL; -- INCORRECT
SELECT * FROM employes WHERE manager_id IS NULL; -- CORRECT

-- Gestion des NULL dans les calculs
SELECT 
    nom,
    COALESCE(bonus, 0) as bonus_final,
    salaire + COALESCE(bonus, 0) as salaire_total
FROM employes;
```

### Q16: UNION vs UNION ALL

```sql
-- UNION (élimine les doublons - plus lent)
SELECT nom FROM employes_actifs
UNION
SELECT nom FROM employes_anciens;

-- UNION ALL (garde les doublons - plus rapide)
SELECT nom FROM employes_actifs
UNION ALL
SELECT nom FROM employes_anciens;
```

### Q17: Différence entre DELETE, TRUNCATE, DROP

```sql
-- DELETE: supprime des lignes (peut être rollback, déclenche triggers)
DELETE FROM employes WHERE dept_id = 5;

-- TRUNCATE: vide la table (plus rapide, reset auto-increment)
TRUNCATE TABLE employes;

-- DROP: supprime la structure complète
DROP TABLE employes;
```

## 7. Exercices Pratiques Avancés

### Q18: Séquences consécutives

```sql
-- Trouver les dates consécutives de connexion
WITH connexions_numerotees AS (
    SELECT 
        user_id,
        date_connexion,
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY date_connexion) as rn,
        date_connexion - INTERVAL ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY date_connexion) DAY as groupe
    FROM connexions
)
SELECT 
    user_id,
    MIN(date_connexion) as debut_sequence,
    MAX(date_connexion) as fin_sequence,
    COUNT(*) as jours_consecutifs
FROM connexions_numerotees
GROUP BY user_id, groupe
HAVING COUNT(*) >= 7; -- 7 jours consécutifs ou plus
```

### Q19: Pivot dynamique

```sql
-- Transformer lignes en colonnes
SELECT 
    dept_id,
    SUM(CASE WHEN EXTRACT(MONTH FROM date_embauche) = 1 THEN 1 ELSE 0 END) as janvier,
    SUM(CASE WHEN EXTRACT(MONTH FROM date_embauche) = 2 THEN 1 ELSE 0 END) as fevrier,
    SUM(CASE WHEN EXTRACT(MONTH FROM date_embauche) = 3 THEN 1 ELSE 0 END) as mars
FROM employes
GROUP BY dept_id;
```

### Q20: Calcul de médiane

```sql
-- Médiane avec PERCENTILE_CONT
SELECT 
    dept_id,
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salaire) as mediane_salaire
FROM employes
GROUP BY dept_id;

-- Médiane manuelle
WITH ordered_salaires AS (
    SELECT 
        salaire,
        ROW_NUMBER() OVER (ORDER BY salaire) as rn,
        COUNT(*) OVER () as total_count
    FROM employes
)
SELECT AVG(salaire) as mediane
FROM ordered_salaires
WHERE rn IN ((total_count + 1) / 2, (total_count + 2) / 2);
```

## 8. Questions Conceptuelles

### Q21: Normalisation

- **1NF**: Pas de valeurs multiples dans une colonne
- **2NF**: 1NF + élimination des dépendances partielles
- **3NF**: 2NF + élimination des dépendances transitives

### Q22: ACID

- **Atomicité**: Transaction tout ou rien
- **Cohérence**: Respect des contraintes
- **Isolation**: Transactions indépendantes
- **Durabilité**: Persistance des données

### Q23: Types d'index

- **Clustered**: Détermine l'ordre physique
- **Non-clustered**: Pointeurs vers les données
- **Unique**: Garantit l'unicité
- **Composite**: Sur plusieurs colonnes

## 9. Optimisation Avancée

### Q24: Réécriture de sous-requêtes

```sql
-- Lent: sous-requête corrélée
SELECT * FROM employes e1
WHERE salaire > (
    SELECT AVG(salaire) FROM employes e2 WHERE e2.dept_id = e1.dept_id
);

-- Rapide: JOIN avec CTE
WITH avg_salaires AS (
    SELECT dept_id, AVG(salaire) as avg_sal
    FROM employes
    GROUP BY dept_id
)
SELECT e.*
FROM employes e
JOIN avg_salaires a ON e.dept_id = a.dept_id
WHERE e.salaire > a.avg_sal;
```

### Q25: Pagination efficace

```sql
-- Pagination avec OFFSET (lent sur gros volumes)
SELECT * FROM employes ORDER BY id LIMIT 20 OFFSET 1000;

-- Pagination par curseur (plus efficace)
SELECT * FROM employes WHERE id > 1000 ORDER BY id LIMIT 20;
```

## 10. Conseils pour l'entretien

### Préparation
- **Réviser les concepts fondamentaux** : JOINs, GROUP BY, sous-requêtes
- **Pratiquer les window functions** : ROW_NUMBER, RANK, LAG/LEAD
- **Maîtriser l'optimisation** : index, plans d'exécution, réécriture de requêtes
- **Connaître les pièges courants** : NULL, UNION vs UNION ALL, WHERE vs HAVING

### Pendant l'entretien
- **Comprendre le problème** avant de coder
- **Partir du simple** puis optimiser
- **Expliquer votre raisonnement** à haute voix
- **Tester avec des exemples** concrets
- **Proposer plusieurs solutions** quand c'est possible

### Questions à poser
- **Volume de données** attendu ?
- **Fréquence d'exécution** de la requête ?
- **Contraintes de performance** spécifiques ?
- **Structure exacte** des tables ?

### Erreurs à éviter
- Ne pas gérer les **valeurs NULL**
- Oublier les **index** sur les colonnes de jointure
- Utiliser **SELECT *** sans raison
- Ne pas considérer les **cas limites**
- Ignorer l'**optimisation** pour de gros volumes

---
```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    nom VARCHAR(255),
    prenom VARCHAR(255),
    CONSTRAINT uix_nom_prenom UNIQUE (nom, prenom)
);
```

*Ce guide couvre les aspects essentiels des entretiens techniques SQL. Pratiquez régulièrement avec des données réelles pour maîtriser ces concepts.*