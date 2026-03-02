---
description: Register inputs for a new agent and let Maniac bootstrap completions and build a model.
---

# Building New Agents

Most Maniac workflows start with an existing LLM call — you capture production traffic, then optimize. But what if you're building something new? You know the kinds of requests your agent will handle, but you don't have completions yet.

Maniac supports **input-only registration**. You register the inputs your agent needs to handle — without any outputs — and Maniac's infrastructure generates completions, bootstraps a training dataset, and builds an optimized model for you.

This is especially useful for **high-throughput agents** where you can enumerate or sample the input space upfront.

***

## How It Works

1. **Create a container** with a base model and system prompt that describe the agent you want to build.
2. **Register inputs** — just the messages, no `output` field needed.
3. **Maniac handles the rest** — it generates completions, bootstraps a training dataset, and automatically optimizes a model for your task. No manual steps required.

***

## Step 1 — Create a Container

Set the `initial_model` to the frontier model you want Maniac to bootstrap from, and the `default_system_prompt` to the instructions for your new agent.

{% tabs %}
{% tab title="Python" %}
```python
from maniac import Maniac

maniac = Maniac()

container = maniac.containers.create(
    label="ticket-router",
    initial_model="openai/gpt-4o",
    default_system_prompt=(
        "You are a support ticket router. Given a customer message, "
        "respond with exactly one label: billing, technical, account, or general."
    ),
)
```
{% endtab %}

{% tab title="TypeScript" %}
```typescript
import Maniac from "maniac-js";

const maniac = new Maniac();

const container = await maniac.containers.create({
  label: "ticket-router",
  initial_model: "openai/gpt-4o",
  default_system_prompt:
    "You are a support ticket router. Given a customer message, " +
    "respond with exactly one label: billing, technical, account, or general.",
});
```
{% endtab %}
{% endtabs %}

***

## Step 2 — Register Inputs Only

Each item only needs an `input` — the `output` field is optional. When omitted, Maniac treats the entry as an input that needs a completion generated.

{% tabs %}
{% tab title="Python" %}
```python
inputs = [
    "I was charged twice for my last order",
    "The app crashes when I try to upload a file",
    "How do I change my email address?",
    "Do you offer student discounts?",
]

maniac.chat.completions.register(
    container="ticket-router",
    items=[
        {
            "input": {
                "messages": [{"role": "user", "content": text}]
            }
        }
        for text in inputs
    ],
)
```
{% endtab %}

{% tab title="TypeScript" %}
```typescript
const inputs = [
  "I was charged twice for my last order",
  "The app crashes when I try to upload a file",
  "How do I change my email address?",
  "Do you offer student discounts?",
];

await maniac.chat.completions.register({
  container: "ticket-router",
  items: inputs.map((text) => ({
    input: {
      messages: [{ role: "user", content: text }],
    },
  })),
});
```
{% endtab %}
{% endtabs %}

That's it — no `output` field, no completions to generate yourself.

***

## Uploading Inputs at Scale

If you have a large corpus of representative inputs (e.g. historical support tickets, logged queries, synthetic samples), upload them in batches to avoid timeouts.

{% tabs %}
{% tab title="Python" %}
```python
BATCH_SIZE = 1500

for i in range(0, len(all_inputs), BATCH_SIZE):
    batch = all_inputs[i : i + BATCH_SIZE]

    maniac.chat.completions.register(
        container="ticket-router",
        items=[
            {
                "input": {
                    "messages": [{"role": "user", "content": text}]
                }
            }
            for text in batch
        ],
    )

    print(f"Uploaded {i + len(batch)} / {len(all_inputs)}")
```
{% endtab %}

{% tab title="TypeScript" %}
```typescript
const BATCH_SIZE = 1500;

for (let i = 0; i < allInputs.length; i += BATCH_SIZE) {
  const batch = allInputs.slice(i, i + BATCH_SIZE);

  await maniac.chat.completions.register({
    container: "ticket-router",
    items: batch.map((text) => ({
      input: {
        messages: [{ role: "user", content: text }],
      },
    })),
  });

  console.log(`Uploaded ${i + batch.length} / ${allInputs.length}`);
}
```
{% endtab %}
{% endtabs %}

***

## What Happens Next

Once your inputs are registered, add an **Eval** from the **Evals** tab inside your container to define how the agent's outputs should be judged (judge prompt or code eval). From there, Maniac takes over automatically — it generates completions from your base model, builds a training dataset, and optimizes a new model in the background. When a model is ready, it will appear in the **Models** tab inside your container, where you can deploy it so inference requests are routed through it.

{% hint style="info" %}
The more representative your inputs are of real production traffic, the better the resulting model will perform. If you can, sample inputs that cover the full range of scenarios your agent will encounter.
{% endhint %}
