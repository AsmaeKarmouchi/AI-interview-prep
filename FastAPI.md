#  Guide FastAPI – Bases, Validation, Dépendances

Ce document présente les bases de **FastAPI**, l'utilisation de **Pydantic** pour la validation, et la gestion des dépendances avec **Depends**.
d'après plusieurs vd youtube

---

# 1.  Les bases de FastAPI

```python
from fastapi import FastAPI

app = FastAPI()  # Création de l'application FastAPI

# Endpoint racine
@app.get("/")  # Définition d'une route GET pour la racine
async def read_root():  # Fonction asynchrone pour gérer la requête
    return {"Hello": "World"}  # Réponse JSON

# Endpoint avec paramètre de chemin
@app.get("/items/{item_id}") #ici item_id est un paramètre de chemin 
async def read_item(item_id: int, q: str = None):
    result = await db.fetch_data(item_id)  # Appel async vers la base de données pour récupérer des données
    return {"item_id": item_id, "q": q, "data": result}
```

### Pourquoi `async/await` est important ?

* Permet des opérations **non bloquantes**.
* Gère **des milliers de requêtes concurrentes**.
* Idéal pour les tâches **I/O-bound** : API, DB, réseaux…
* Ne bloque pas l'event loop en attendant une opération lente.
* Réduit la **latence** et améliore la **réactivité**.

---

# 2.  Pydantic : Validation & Sérialisation

FastAPI utilise **Pydantic** pour valider automatiquement les données d'entrée et convertir les réponses.

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class User(BaseModel):  # Définition d'un modèle Pydantic
    id: int
    name: str
    email: str

@app.post("/users")
def create_user(user: User):  # Validation automatique
    return user
```

###  Avantages de Pydantic

* Vérifie automatiquement les types (ex : `id` doit être un `int`).
* Retourne une **erreur 422** si les données sont invalides.
* Transforme le JSON → objet Python.
* Sérialise automatiquement l'objet Python → JSON en réponse.

Exemple invalide :

```json
{"id": "abc", "name": "Tom"}
```

 Rejet automatique car `id` n'est pas un entier.

---

# 3.  Gestion des dépendances avec `Depends`

FastAPI propose un système puissant d'injection de dépendances pour organiser votre code.

```python
from fastapi import Depends


def get_db():
    db = DatabaseConnection()
    try:
        yield db  # Fournit la ressource 
    finally:
        db.close()  # Nettoyage

@app.get("/users")
def list_users(db = Depends(get_db)): #ici on injecte la dépendance connexion db 
    return db.get_users()
```

###  Avantages des dépendances

* Injection simple de services : DB, configuration, API externes…
* Meilleure **réutilisation du code** et séparation des responsabilités.
* Tests facilités : possibilité de remplacer une dépendance.

  ```python
  app.dependency_overrides[get_db] = lambda: FakeDB()
  ```
* **Mise en cache automatique** des dépendances pendant une requête.

---

#  Résumé

FastAPI offre :

* Syntaxe simple et performante grâce à `async/await`.
* Validation automatique avec **Pydantic**.
* Gestion claire des services via **Depends** et l'injection de dépendances.
* API rapides, fiables et faciles à maintenir.


# 4.  Paramètres & Mise en cache des dépendances

```python
def get_settings():
    print("Settings loaded")
    return Settings()
```

Si plusieurs dépendances utilisent `Depends(get_settings)`, FastAPI :

* n'exécute **qu’une seule fois** `get_settings` par requête
* réutilise le même objet

---

# 5.  HTTP Methods & Routing

* Support complet de **GET, POST, PUT, DELETE, PATCH**, etc.
* Organisation modulaire avec **APIRouter**.
* Gestion claire des erreurs avec **HTTPException**.

---

# 6. Middlewares et Sécurité dans FastAPI

###  Middlewares intégrés

FastAPI supporte :

* **CORS**
* **GZip**
* **Request Logging**
* **Middlewares personnalisés**

### Exemple : CORS  (Cross-Origin Resource Sharing)
- Used when your frontend (React, Vue, Angular…) and backend run on different domains.
- Lets your frontend talk to your API safely.

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### Exemple : Compression GZip
- Compresses responses to reduce bandwidth.
- Faster responses, smaller payloads.

```python
from fastapi.middleware.gzip import GZipMiddleware
app.add_middleware(GZipMiddleware, minimum_size=1000)
```
- Automatically compresses responses larger than 1000 bytes.

### Exemple : Middleware custom
- You can create your own middleware for logging, authentication checks, timers, etc.

```python
from fastapi import Request

@app.middleware("http")
async def log_requests(request: Request, call_next):
    print("Incoming:", request.url)
    response = await call_next(request)
    return response
```

---

# 7.  Sécurité : OAuth2, JWT & Auth

- jwt (JSON Web Tokens) Used for stateless authentication. for secure login sessions in APIs.

```python
from fastapi.security import OAuth2PasswordBearer

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="login")


def get_current_user(token: str = Depends(oauth2_scheme)):
    return decode_jwt(token)
```

FastAPI facilite :

* lecture du token
* validation du JWT
* récupération de l’utilisateur courant

---

# 8.  Documentation Automatique

FastAPI utilise **OpenAPI** & **JSON Schema**.
Généré automatiquement :

* `/docs` → Swagger UI
* `/redoc` → ReDoc

Permet d'explorer l’API et tester les endpoints.

---

# 9.  Bases de données & Outils

* **Alembic** pour migrations SQL.
* **Beanie** pour MongoDB.
* **Tortoise ORM** pour plusieurs DB NoSQL.
* **Moto** pour mock AWS (S3, DynamoDB…).

Déploiement souvent via :

* **Uvicorn** (ASGI)
* **Hypercorn** (ASGI)

---

#  Gunicorn vs Uvicorn dans FastAPI

###  Uvicorn

* Serveur **ASGI** (asynchrone)
* Optimisé pour FastAPI
* Supporte WebSockets, async I/O
* Très rapide

###  Gunicorn

* Serveur **WSGI** (synchrone) mais peut lancer Uvicorn via des workers ASGI
* On l’utilise souvent avec :

  ```bash
  gunicorn -k uvicorn.workers.UvicornWorker app:app
  ```
* Gère plusieurs workers & processus
* Bonne stabilité pour production

###  Différence clé

* **Uvicorn seul** = ultra rapide, async, idéal pour FastAPI
* **Gunicorn + UvicornWorker** = meilleure gestion multi‑processus pour fortes charges

---
