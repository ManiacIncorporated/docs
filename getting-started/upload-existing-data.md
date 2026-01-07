---
description: Adding pre-existing data to a container
---

# Upload Existing Datasets

Maniac lets you upload existing datasets directly into a container . These might be inference logs from a different inference provider, or a labeled dataset. Once uploaded, the data appears alongside inference logs and can be used for optimization and evaluation.

## Boilerplate

```python
from maniac import Maniac

client = Maniac()

container = client.containers.create(
    label="my-container",
    initial_model="some-base-model",
    initial_system_prompt="Your system prompt here",
)

dataset = [
    {
        "input": {
            "messages": [
                {"role": "system", "content": "..."},
                {"role": "user", "content": "..."},
            ]
        },
        "output": {
            "choices": [
                {"message": {"role": "assistant", "content": "..."}}
            ]
        },
    }
]

client.chat.completions.register(
    container=container,
    dataset=dataset,
)

```

Each dataset entry consists of:

* `input` : the messages sent to the model
* `output` : the expected assistant response

## Example: Uploading a HuggingFace Dataset

#### Prerequisites

```python
export MANIAC_API_KEY=...
pip install maniac datasets
```

#### Load the HuggingFace Dataset

```python
from datasets import load_dataset

DATASET = "coastalchp/ledgar"

ds = load_dataset(DATASET)
train = ds["train"]

clauses = train["text"]
labels = train.features["label"].names
y = train["label"]
```

#### Define the System Prompt

```python
system_prompt = (
    "You are a legal clause classifier.\n"
    "Given a clause, return exactly one label from this list:\n"
    + "\n".join(f"- {l}" for l in labels)
    + "\nRespond with only the label name."
)
```

#### Create a container

You can also skip this step and upload a dataset to an existing container, where it will be combined with any existing inference logs.

```python
from maniac import Maniac

client = Maniac()

container = client.containers.create(
    label="ledgar-register",
    initial_model="openai/gpt-4o-mini",
    initial_system_prompt=system_prompt,
)
```

#### Upload Data in Batches

For large datasets, it's recommended uploading in batches to avoid timeouts.&#x20;

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
                            "content": labels[y[i]],
                        }
                    }
                ]
            },
        })

    client.chat.completions.register(
        container=container,
        dataset=dataset,
    )

    print(f"Uploaded samples {batch_start}–{batch_end - 1}")

```
