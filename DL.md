
# Propagation avant (Forward Pass)

Les données d’entrée traversent successivement les couches du réseau.

Chaque neurone calcule : somme pondérée + biais → fonction d’activation (ReLU, sigmoïde, etc.) pour introduire une non-linéarité dans le modèle.

La couche de sortie (Output Layer) produit la prédiction finale :
- **Classification** → probabilités (Softmax)
- **Régression** → valeur numérique

> Sert à transformer les données brutes en prédictions.


# Propagation arrière (Backward Pass / Backpropagation)

1. Compare la prédiction à la valeur réelle via une fonction de perte.
2. Calcule les gradients (dérivées partielles de la perte par rapport aux poids). Ce gradient indique la contribution de chaque paramètre à l'erreur.
3. Met à jour les paramètres grâce à un optimiseur (ex. SGD, Adam).
4. Répété sur plusieurs époques → le réseau apprend en réduisant l’erreur.

> C’est la phase d’apprentissage.


# Pourquoi c’est important ?

- **Optimisation continue** : Forward = prédiction, Backward = correction.
- **Amélioration des performances** : les mises à jour itératives réduisent l’erreur.
- **Résolution de problèmes complexes** : les couches cachées + non-linéarités capturent des motifs difficiles.
- **Base de l’IA moderne** : fondement du deep learning, des LLM, de la vision par ordinateur, etc.


## Exemple en PyTorch

```python
import torch
import torch.nn as nn
import torch.optim as optim

# Données (x, y = 2x)
X = torch.tensor([[1.0], [2.0], [3.0], [4.0]])
Y = torch.tensor([[2.0], [4.0], [6.0], [8.0]])

# Modèle simple : une couche linéaire
model = nn.Linear(in_features=1, out_features=1)

# Fonction de perte et optimiseur (SGD = descente de gradient stochastique)
criterion = nn.MSELoss()
optimizer = optim.SGD(model.parameters(), lr=0.01)

# Training loop
for epoch in range(100):
    model.train()                          # mode entraînement
    # Forward pass
    y_pred = model(X)
    loss = criterion(y_pred, Y)
    # Backward pass
    optimizer.zero_grad()                  # remettre les gradients à zéro
    loss.backward()                        # calcul des gradients
    optimizer.step()                       # mise à jour des poids (gradient descent)
    if (epoch+1) % 20 == 0:
        print(f"Epoch {epoch+1}, Loss: {loss.item():.4f}")

# Test du modèle
with torch.no_grad():                       # pas besoin de calculer les gradients en inference
    test = torch.tensor([[5.0]])
    print("Prediction for x=5:", model(test).item())
```



## Training Loop (Boucle d’entraînement)

Étapes répétées pendant plusieurs epochs :

1. Forward pass → prédictions
2. Calcul de la loss
3. Backward pass → calcul des gradients
4. Optimizer.step() → mise à jour des poids

```python
for epoch in range(epochs):
    y_pred = model(x)
    loss = loss_fn(y_pred, y)
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()
```


## Hyperparamètres

Paramètres fixés avant l’entraînement :

- **Learning rate** : vitesse d’apprentissage
- **Batch size** : nombre d’exemples vus avant une mise à jour des poids
- **Nombre d’epochs** : combien de fois l’ensemble des données est vu
- **Architecture du modèle** (profondeur, largeur) : nombre de couches, neurones, type d’activations

Ont un impact majeur sur les performances.

> Leur tuning = étape clé pour un bon modèle.





## Batch Normalization

- **Pourquoi ?**
    - Stabilise et accélère l’entraînement
    - Évite que les activations explosent ou disparaissent
    - Agit comme une forme de régularisation
- **Comment ?**
    - Normalise les activations couche par couche puis applique une transformation affine (γ, β)

```python
nn.Sequential(
    nn.Linear(128, 64),
    nn.BatchNorm1d(64),
    nn.ReLU()
)
```


## Overfitting / Underfitting

- **Overfitting** : modèle trop complexe, apprend par cœur, généralise mal
- **Underfitting** : modèle trop simple, n’apprend pas assez

> On vise le juste milieu (bon sur entraînement et validation). Objectif : équilibre entre biais et variance.



## MLP (Perceptron multicouche)

Un réseau de neurones feedforward composé de plusieurs couches entièrement connectées (fully connected layers) :
- Une couche d’entrée (features)
- Plusieurs couches cachées avec fonctions d’activation non linéaires
- Une couche de sortie (classification ou régression)

Ne capture pas bien la structure spatiale (images) ou temporelle (séquences).

```python
mlp = nn.Sequential(
    nn.Linear(10, 32),
    nn.ReLU(),
    nn.Linear(32, 2)
)
```

lstm = nn.LSTM(input_size=5, hidden_size=10, batch_first=True)

## RNN / LSTM / GRU (Séquences)

### RNN (Recurrent Neural Network)
- Réseau conçu pour traiter des séquences de données (texte, séries temporelles, audio)
- Les neurones ont des connexions récurrentes qui gardent une mémoire des étapes précédentes
- Les RNN classiques souffrent du vanishing / exploding gradient (perte de mémoire sur les longues séquences)

### LSTM (Long Short-Term Memory)
- Variante de RNN avec des cellules de mémoire et des portes (input, forget, output)
- Capable de conserver l’information sur de longues séquences
- Très utilisé en NLP, speech recognition, séries temporelles

```python
lstm = nn.LSTM(input_size=5, hidden_size=10, batch_first=True)
seq = torch.randn(2, 7, 5)  # batch=2, séquence=7, features=5
out, (h, c) = lstm(seq)
```

### GRU (Gated Recurrent Unit)
- Version simplifiée du LSTM
- Moins de portes → plus rapide à entraîner, mais souvent aussi performant


## CNN (Convolutional Neural Network)

```python
cnn = nn.Sequential(
    nn.Conv2d(3, 16, kernel_size=3, stride=1, padding=1),
    nn.ReLU(),
    nn.MaxPool2d(2)
)
```

## Transformers = NLP + vision moderne

## Régularisation + BatchNorm

### Régularisation
Techniques pour éviter l’overfitting :
- **Dropout** : désactive aléatoirement certains neurones
- **L1 / L2 regularization** : pénalise les grands poids
- **Early stopping** : arrêter l’entraînement quand la validation cesse de s’améliorer
- **Data augmentation** : enrichir artificiellement les données (images, texte, etc.)



## Équilibrer les données (Data balancing)

**Problème** : classes déséquilibrées → biais du modèle

**Solutions** :
- Oversampling de la classe minoritaire
- Undersampling de la classe majoritaire
- Class weights dans la loss function
- Data augmentation spécifique à la classe minoritaire



## Comment faire du finetuning : étapes détaillées

```python
import torch
import torch.nn as nn
from torch.utils.data import DataLoader, TensorDataset
import torch.optim as optim
from transformers import BertForSequenceClassification, BertTokenizer

# 1. Charger le modèle et le tokenizer
model_name = "bert-base-uncased"
num_classes = 3
tokenizer = BertTokenizer.from_pretrained(model_name)
model = BertForSequenceClassification.from_pretrained(model_name, num_labels=num_classes)

# 2. Gel des couches de base (toutes sauf la classification)
for param in model.bert.parameters():
    param.requires_grad = False
# La dernière couche classifier sera entraînée
for param in model.classifier.parameters():
    param.requires_grad = True

# 3. Préparer les données (exemple minimal)
texts = ["I love machine learning", "Transformers are powerful", "BERT is great"]
labels = [0, 1, 2]
encoding = tokenizer(texts, padding=True, truncation=True, return_tensors="pt")
input_ids = encoding['input_ids']
attention_mask = encoding['attention_mask']
labels = torch.tensor(labels)
dataset = TensorDataset(input_ids, attention_mask, labels)
dataloader = DataLoader(dataset, batch_size=2, shuffle=True)

# 4. Définir loss et optimiseur
criterion = nn.CrossEntropyLoss()
optimizer = optim.Adam(model.classifier.parameters(), lr=2e-5)  # uniquement dernière couche

# 5. Boucle d'entraînement
model.train()
for epoch in range(3):  # 3 epochs pour l'exemple
    for batch in dataloader:
        input_ids, attention_mask, labels_batch = batch
        optimizer.zero_grad()
        outputs = model(input_ids=input_ids, attention_mask=attention_mask)
        loss = criterion(outputs.logits, labels_batch)
        loss.backward()
        optimizer.step()
    print(f"Epoch {epoch+1}, Loss: {loss.item():.4f}")

# 6. Évaluation simple
model.eval()
with torch.no_grad():
    outputs = model(input_ids=input_ids, attention_mask=attention_mask)
    predictions = torch.argmax(outputs.logits, dim=1)
    print("Predictions:", predictions.tolist())
```




## LoRA (Low-Rank Adaptation)

Permet de finetuner seulement une petite partie du modèle en insérant des matrices de faible rang dans certaines couches (souvent les poids des projections dans l’attention).

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
from peft import LoraConfig, get_peft_model, TaskType
import torch

# 1. Charger le modèle pré-entraîné
model_name = "gpt2"
model = AutoModelForCausalLM.from_pretrained(model_name)
tokenizer = AutoTokenizer.from_pretrained(model_name)

# 2. Configurer LoRA
lora_config = LoraConfig(
    r=8,                 # rang faible
    lora_alpha=16,       # facteur de mise à l’échelle
    target_modules=["c_attn"],  # les modules à adapter
    lora_dropout=0.1,
    task_type=TaskType.CAUSAL_LM
)

# 3. Appliquer LoRA au modèle
model = get_peft_model(model, lora_config)

# 4. Préparer les données
texts = ["Hello world", "Fine-tuning LoRA is fun"]
inputs = tokenizer(texts, return_tensors="pt", padding=True, truncation=True)

# 5. Définir optimiseur et loss
optimizer = torch.optim.AdamW(model.parameters(), lr=1e-4)
loss_fn = torch.nn.CrossEntropyLoss()

# 6. Boucle d’entraînement simplifiée
model.train()
for epoch in range(3):
    optimizer.zero_grad()
    outputs = model(**inputs, labels=inputs["input_ids"])
    loss = outputs.loss
    loss.backward()
    optimizer.step()
    print(f"Epoch {epoch+1}, Loss: {loss.item():.4f}")

# 7. Évaluation simple
model.eval()
with torch.no_grad():
    outputs = model.generate(tokenizer("Fine-tuning ", return_tensors="pt")["input_ids"], max_length=20)
    print(tokenizer.decode(outputs[0]))
```
