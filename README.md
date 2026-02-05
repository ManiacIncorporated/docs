---
layout:
  width: default
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
---

# Maniac: Your best model in one click.

Maniac is an enterprise AI platform that makes it easy to replace existing LLM API calls with fine-tuned, task-specific models. Drop Maniac in with one line of code to:

* Capture and structure production **LLM traffic**
* Automatically **fine-tune** and **evaluate** Small Language Models (SLMs) on your tasks
* **Replace over-generalized LLM calls** with higher performance, lower latency models built for just what you need
* **Focus engineering time where it matters most:** building and refining high-quality model evaluations—not managing infrastructure, hyperparameters, or bespoke fine-tuning pipelines

All with virtually no changes to your existing codebase.

## Getting started

{% stepper %}
{% step %}
#### Sign up for Maniac

Head over to [https://app.maniac.ai/auth/register](https://app.maniac.ai/auth/register)
{% endstep %}

{% step %}
#### **Create a new Organization**

Organizations house multiple projects.&#x20;
{% endstep %}

{% step %}
#### Add a Project

All your work — containers, evals, and deployments — live here.
{% endstep %}

{% step %}
#### **Generate an API key**

From your project settings
{% endstep %}
{% endstepper %}

{% embed url="https://youtu.be/gTcfZSXy4hM" %}

***

## Dropping Maniac into your Codebase

#### Set environment variables

```python
export MANIAC_API_KEY="your-api-key"
export MANIAC_BASE_URL="https://platform.maniac.ai/api/v1
```

Containers log inference and automatically build datasets for fine-tuning and evaluation. <mark style="color:$info;">`initial_model`</mark> sets the model used in that container until a Maniac model is deployed in its place.

#### Create a container

```python
import os
import requests

BASE_URL = os.getenv("MANIAC_BASE_URL")
API_KEY = os.getenv("MANIAC_API_KEY")

headers = {
    "Authorization": f"Bearer {API_KEY}",
    "Content-Type": "application/json",
}

response = requests.post(
    f"{BASE_URL}/containers",
    headers=headers,
    json={
        "label": "my-container",
        "initial_model": "openai/gpt-5.2",
        "api": "chat.completions",
        "default_system_prompt": "You are a helpful math tutor.",
    },
)

response.raise_for_status()
container = response.json()

print("Created container:", container["label"])
```

#### Generating Inference Logs

Now that you've made a container, let's add some data to it. You can either register completions with any existing provider, or run inference through Maniac. In both cases, inference logs will auto-populate in your container. External datasets can also be [manually uploaded](datasets/register-completions.md).&#x20;

{% tabs %}
{% tab title="Registering Completions" %}
```python
import os
import requests

BASE_URL = os.getenv("MANIAC_BASE_URL")
API_KEY = os.getenv("MANIAC_API_KEY")

headers = {
    "Authorization": f"Bearer {API_KEY}",
    "Content-Type": "application/json",
}

_input = {
  "model": "openai/gpt-5"
  "messages": [{
    "role": "user",
    "content": "Hello!"
  }]
}

_output = some_other_provider.chat.completions.create(**params)

requests.post(
    f"{BASE_URL}/chat/completions/register",
    headers=headers,
    json={
        "container": "my-container",
        "items": [
            "input": _input,
            "output": _output
        ],
    },
)
```
{% endtab %}

{% tab title="Running Inference Through Maniac" %}
```python
response = requests.post(
    f"{BASE_URL}/chat/completions",
    headers=headers,
    json={
        "model": "maniac:my-container",
        "messages": [
            {
                "role": "user",
                "content": "A train travels 120 miles in 2 hours. What is its average speed?"
            }
        ],
    },
)

response.raise_for_status()
completion = response.json()

print(response["choices"][0]["message"]["content"])
# Output: "The average speed is 60 miles per hour. This is calculated by dividing distance (120 miles) by time (2 hours): 120 ÷ 2 = 60 mph."
```
{% endtab %}
{% endtabs %}

> **Note:** We recommend defining the system prompt at the **container level**. All inference requests executed through that container will automatically inherit this system prompt. If a request’s `messages` array includes its own system prompt, it will override the container-level system prompt for that request only.

{% embed url="https://youtu.be/5ygzMi4okJ8" %}

***

## Optimizing your model

The inference logs in your container now serve as training data for a new SLM—fully yours, lower latency, cheaper, and optimized specifically for your task.

{% stepper %}
{% step %}
### Create an <mark style="color:green;">Eval</mark>

Evaluations define the optimization target. They can be implemented as arbitrary code or defined using judge prompts.

From the **Evals** tab inside a container, **Add Eval**.

{% tabs %}
{% tab title="Judge Eval" %}
<div data-with-frame="true"><figure><img src=".gitbook/assets/Screenshot 2026-01-06 at 12.43.14 PM.png" alt=""><figcaption></figcaption></figure></div>
{% endtab %}

{% tab title="Code Eval" %}
<div data-with-frame="true"><figure><img src=".gitbook/assets/Screenshot 2026-01-06 at 1.15.10 PM.png" alt=""><figcaption></figcaption></figure></div>


{% endtab %}
{% endtabs %}
{% endstep %}

{% step %}
### Launch <mark style="color:green;">Optimization</mark>

Once you've defined an eval, the **Optimization** dashboard lets you configure and run post-training pipelines using techniques such as SFT, GRPO, and GEPA.&#x20;

Each stage of the pipeline is modular, allowing you to select base models, the evaluation to optimize against, adjust hyperparameters, swap classifier heads, and experiment with different training strategies.

{% embed url="https://youtu.be/O07rVLiWMR4" %}
{% endstep %}
{% endstepper %}

***

## Deploy.

Optimized models can be be deployed into a container from the **Models** tab. Once deployed, you can chat with your generated models, and inference requests are now routed through the Maniac model instead of the <mark style="color:$info;">`initial_model`</mark>.

<div data-with-frame="true"><figure><img src=".gitbook/assets/Screenshot 2026-01-06 at 2.05.29 PM.png" alt=""><figcaption></figcaption></figure></div>

### Need help?

:e-mail: Email us at support@maniac.ai

We'll get back to you within a day.
