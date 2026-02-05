---
description: Adding pre-existing data to a container.
---

# Register Completions

Maniac lets you direct existing LLM traffic from a different inference provider into a container or upload a static dataset. Once uploaded, these logs can be used for optimization and evaluation.

## Example: Uploading a HuggingFace Dataset

Let's walk through an example of uploading a static dataset using [LEDGAR](https://aclanthology.org/2020.lrec-1.155.pdf) (Tuggener et al. 2020), great for training and testing legal classification models.&#x20;

#### Prerequisites

```python
export MANIAC_API_KEY=...
```

#### Load the HuggingFace Dataset

```python
from datasets import load_dataset

DATASET = "coastalchp/ledgar"

# Load dataset
dataset = load_dataset(DATASET)
train_split = dataset["train"]

# Extract inputs and labels
clauses = train_split["text"]                  
label_ids = train_split["label"]              
label_names = train_split.features["label"].names
```

#### Define the System Prompt

```python
system_prompt = (
    "You are a legal clause classifier.\n"
    "Given a clause, return exactly one label from this list:\n"
    + "\n".join(f"- {label}" for label in label_names)
    + "\nRespond with only the label name."
)
```

#### Create a container

You can also skip this step and upload a dataset to an existing container, where it will be combined with any existing inference logs.

```python
import requests

response = requests.post(
    f"{BASE_URL}/containers",
    headers=headers,
    json={
        "label": "ledgar-container",
        "initial_model": "openai/gpt-5.2",
        "api": "chat.completions",
        "default_system_prompt": system_prompt,
    },
)

response.raise_for_status()
container = response.json()
```

#### Upload Data in Batches

For large datasets, it's recommended uploading in batches to avoid timeouts. Each dataset entry consists of an input and output in chat completions format, as well as optional metadata.&#x20;

```python
BATCH_SIZE = 1500
START = 0
MAX_SAMPLES = 60000

end = min(len(clauses), START + MAX_SAMPLES)

for batch_start in range(START, end, BATCH_SIZE):
    batch_end = min(batch_start + BATCH_SIZE, end)
    dataset = []

    for i in range(batch_start, batch_end):
        messages = [
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": clauses[i]},
        ]

        dataset.append({
            "input": {"messages": messages},
            "output": {
                "choices": [
                    {
                        "message": {
                            "role": "assistant",
                            "content": label_names[label_ids[i]],
                        }
                    }
                ]
            },
        })

    requests.post(
        f"{BASE_URL}/chat/completions/register",
        headers=headers,
        json={
            "container": "ledgar-container",
            "items": dataset
        },
    )

    print(f"Uploaded samples {batch_start}–{batch_end - 1}")
```

> **Note:** Unlike generating completions inside a container—where the container’s system prompt is automatically applied—**registering (logging) existing completions requires the system prompt to be included explicitly with each messages object**. Registered completions do not inherit the container-level system prompt.
