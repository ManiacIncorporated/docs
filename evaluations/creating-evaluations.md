---
description: Code and judge evals for your Maniac models.
---

# Creating Evaluations

Evaluations (“evals”) define what Maniac uses to guide optimization.

Maniac supports two types of evaluations:

1. **Judge prompt evaluations**
2. **Code evaluations**

## Judge Prompt Evals

Judge prompt evals compare two candidate outputs—**Response A** and **Response B**—for the same input. Maniac runs these comparisons in a tournament-style setup to determine which model performs better.

In Maniac’s tournament setup:

* **Response A** is produced by the **candidate model being evaluated**
* **Response B** is produced by a **reference source**, such as a frontier LLM or labeled data uploaded into your container.&#x20;

The judge will returns <mark style="color:green;background-color:green;">`TRUE`</mark> if Response A is at least as good as Response B, and <mark style="color:red;background-color:red;">`FALSE`</mark> otherwise.&#x20;

For example:

`Is response A at least as good as response B in generating a headline for the provided news snippet? Return either True or False.`

***

## Code Evals

Code-based evals give you full control over how outputs are scored. These evals are written as Python functions and operate on a single sample at a time.

A code eval must:

* Accept a single `item` argument
* Return a dictionary with a numeric `score`
* Use `item["sample"]` and `item["ground_truth"]` as inputs

#### Item Structure

```python
item = {
    "sample": {
        "input":  chat_completion_request,
        "output": chat_completion_response
    },
    "ground_truth": {
        "input":  chat_completion_request,
        "output": chat_completion_response
    }
}
```

#### Dependencies

Code evaluations can depend on any external Python packages you need. Just list them in the  `requirements.txt` field when creating the evaluation.

## Examples

### Example: Exact Match

```python
def evaluate(item) -> dict:
    pred = item["sample"]["output"]["choices"][0]["message"]["content"].strip()
    gt = item["ground_truth"]["output"]["choices"][0]["message"]["content"].strip()
    return {"score": 1.0 if pred == gt else 0.0}
```

### Example: Multi-Label IuO (Jaccard Similarity)

```python
def evaluate(item) -> dict:
    def extract(text):
        return {t.strip().lower() for t in text.split(",") if t.strip()}

    pred = extract(item["sample"]["output"]["choices"][0]["message"]["content"])
    gt = extract(item["ground_truth"]["output"]["choices"][0]["message"]["content"])

    if not gt:
        return {"score": 0.0}

    iou = len(pred & gt) / len(pred | gt)
    return {"score": 1.0 if iou >= 0.8 else 0.0}

```

### Example: JSON Schema Validation

```python
import json

def evaluate(item) -> dict:
    text = item["sample"]["output"]["choices"][0]["message"]["content"]

    try:
        obj = json.loads(text)
    except json.JSONDecodeError:
        return {"score": 0.0}

    required_keys = {"title", "summary", "confidence"}
    return {"score": 1.0 if required_keys.issubset(obj.keys()) else 0.0}
```

### Example: Semantic Similarity Evaluation

```python
from sentence_transformers import SentenceTransformer
from sklearn.metrics.pairwise import cosine_similarity
import numpy as np

model = SentenceTransformer("all-MiniLM-L6-v2")

def evaluate(item) -> dict:
    try:
        pred = item["sample"]["output"]["choices"][0]["message"]["content"]
        gt = item["ground_truth"]["output"]["choices"][0]["message"]["content"]
    except Exception:
        return {"score": 0.0}

    # Encode texts
    embeddings = model.encode([pred, gt], normalize_embeddings=True)
    pred_emb, gt_emb = embeddings

    # Compute cosine similarity
    similarity = float(cosine_similarity(
        pred_emb.reshape(1, -1),
        gt_emb.reshape(1, -1)
    )[0][0])

    # Threshold for pass/fail
    return {"score": 1.0 if similarity >= 0.85 else 0.0}
```
