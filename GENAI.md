## Generative AI, Transformers, RAG, Finetuning, LLM - Q/A

### Q1: Qu'est-ce que l'IA générative ?
**Réponse:**
L'IA générative crée de nouveaux contenus (texte, images, audio) à partir de données existantes, souvent via des modèles comme les GANs ou les LLMs.

### Q2: Expliquez le principe des transformers.
**Réponse:**
Les transformers sont des architectures de réseaux de neurones basées sur l'attention, permettant de traiter efficacement des séquences. Ils sont à la base des modèles comme BERT, GPT, T5.
Ils se déclinent en trois architectures principales :
- **Encodeur** : utilisé pour l'analyse (ex : BERT)
Breaks down input text into tokens (like words).Uses self-attention layers to create hidden states that capture the meaning and context of the text.Multiple layers of the encoder are used.
- **Décodeur** : utilisé pour la génération (ex : GPT)
- **Encodeur-Décodeur** : utilisé pour la traduction ou le résumé (ex : T5, BART)

### Composants clés d'un transformer
- **Token** : unité de texte (mot, sous-mot, caractère) transformée en vecteur
- **Embedding** : représentation vectorielle des tokens which capture the semantic meaning of words.
- *Transformer Block*:Each block includes:
- **Multi-Head Self-Attention/ Self-attention** :  It allows tokens to communicate with other tokens, capturing contextual information and relationships between words. 
permet au modèle de pondérer l'importance de chaque token
- - Step 1: Query, Key, and Value Matrices : Each token's embedding vector is transformed into three vectors: Query (Q), Key (K), and Value (V).
Query (Q) "find more information about"
Key (K) It represents the possible tokens the query can attend to.
Value (V) Once we matched the appropriate search term (Query) with the relevant results (Key), we want to get the content (Value) of the most relevant pages.
- - Step 2: Multi-Head Splitting
Query, key, and Value vectors are split into multiple heads, Each head processes a segment of the embeddings independently, capturing different syntactic and semantic relationships. --> parallel learning
- - Step 3: Masked Self-Attention
- - Step 4: Output and Concatenation

- **Feedforward layers / MLP (Multilayer Perceptron) Layer** : couches linéaires pour le traitement des représentations ,  the goal of the MLP is to refine each token's representation.
- **Layer normalization, dropout** : régularisation et stabilisation
- **Output Probabilities**: The final linear and softmax layers transform the processed embeddings into probabilities, enabling the model to make predictions about the next token in a sequence.

### Qu'est-ce qu'un LLM ?
Un LLM (Large Language Model) est un modèle de deep learning entraîné sur de vastes corpus textuels pour comprendre et générer du langage naturel. Il utilise l'architecture des transformers.

### Transformers vs RNN
- **RNN** : traite les séquences de façon itérative, mémoire limitée, difficile à paralléliser
- **Transformers** : traitent toute la séquence en parallèle, capturent mieux les dépendances longues, plus efficaces pour le NLP moderne

### Q3: Qu'est-ce que le Retrieval-Augmented Generation (RAG) ?
**Réponse:**
RAG combine la génération de texte par LLM avec la récupération d'informations à partir d'une base documentaire, améliorant la pertinence et la factualité des réponses.

### Q4: Qu'est-ce que le finetuning d'un LLM ?
**Réponse:**
Le finetuning consiste à adapter un modèle pré-entraîné à une tâche ou un domaine spécifique en le ré-entraînant sur des données ciblées.

### LLM généraliste vs finetuned
- **LLM généraliste** : pré-entraîné sur des données variées, capable de répondre à des tâches générales
- **LLM finetuned** : ré-entraîné sur des données spécifiques pour une tâche ou un domaine précis (ex : classification, Q/A, chatbot métier)

### Méthodes de finetuning
- **Full finetuning** : tous les paramètres du modèle sont mis à jour
- **PEFT (Parameter Efficient Fine-Tuning)** : seules certaines parties du modèle sont adaptées, exemple :
    - **LoRA (Low-Rank Adaptation)** : insère des matrices de faible rang dans les couches d'attention
    - **Entête de classification** : seule la dernière couche (classifier head) est entraînée

## Exemple de code : LoRA avec transformers
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


### Q5: Citez des applications des LLMs en entreprise.
**Réponse:**
Chatbots, assistants virtuels, génération de rapports, recherche documentaire, automatisation de tâches, analyse de sentiment.

## GENAI

**Q: What is Generative AI?**  
- Branch of AI that generates new content (text, image, audio, code).  
- Examples: ChatGPT, Stable Diffusion, MidJourney.

**Q: What is a Language Model (LM) vs Large Language Model (LLM)?**  
- **LM**: Predicts the next token in a sequence.  
- **LLM**: Scaled-up LM with billions of parameters, trained on massive corpora, usually Transformer-based.

**Q: What is RAG (Retrieval-Augmented Generation)?**  
- Combines retrieval (fetch knowledge from database) + generation (LLM creates response).  
- Helps reduce hallucinations.

**Q: What is Fine-tuning in LLMs?**  
- Process of adapting a pre-trained model to a domain/task.  
- Techniques: Full fine-tuning, LoRA, PEFT, instruction tuning, RLHF.

**Q: Evaluation metrics for GenAI?**  
- Text: BLEU, ROUGE, Perplexity.  
- Image: FID, IS.  
- Human feedback increasingly important.
---
