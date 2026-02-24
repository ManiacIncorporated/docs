# Maniac: Your best model in one click.

Maniac is an enterprise AI platform that makes it easy to replace existing LLM API calls with fine-tuned, task-specific models. Drop in Maniac with one line of code to:

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

{% tabs %}
{% tab title="Agentic Setup" %}
Copy this prompt into your AI coding agent (Cursor, Copilot, Claude Code, etc.) and it will instrument your codebase automatically:

> Fetch and follow https://raw.githubusercontent.com/ManiacIncorporated/docs/main/agent-setup.md to instrument this repo for Maniac telemetry.

That's it — the agent will find the best LLM call site, add the Maniac logging hook, and walk you through the changes before applying them.
{% endtab %}

{% tab title="Manual Setup" %}
Follow the steps below to add Maniac to your codebase by hand.
{% endtab %}
{% endtabs %}

#### Install the library

{% tabs %}
{% tab title="Python" %}
```bash
pip install maniac
```
{% endtab %}
{% tab title="TypeScript" %}
```bash
npm install maniac-js
```
{% endtab %}
{% endtabs %}

#### Initialize client

{% tabs %}
{% tab title="Python" %}
```python
from maniac import Maniac

maniac = Maniac(api_key="your-maniac-api-key")
```
{% endtab %}
{% tab title="TypeScript" %}
```typescript
import Maniac from "maniac-js";

const maniac = new Maniac();
```
{% endtab %}
{% endtabs %}

#### Create a container

Containers log inference and automatically build datasets for fine-tuning and evaluation. <mark style="color:red;">`initial_model`</mark> sets the model used in that container until a Maniac model is deployed in its place.

{% tabs %}
{% tab title="Python" %}
```python
container = maniac.containers.create(
  label = "my-container",
  default_system_prompt = "You are a helpful math tutor."
)
```
{% endtab %}
{% tab title="TypeScript" %}
```typescript
const container = await maniac.containers.create({
  label: "my-container",
  initial_model: "openai/gpt-5.2",
});
```
{% endtab %}
{% endtabs %}

#### Log Completions

Now that you've made a container, let's add some data to it.

{% tabs %}
{% tab title="Python" %}
```python
response = maniac.chat.completions.register(
    container = "my-container",
    items = [{
        "input": {
            "messages": [{"role": "user", "content": "Hello!"}]
        },
        "output": {
            "choices": [{
                "message": [{"role": "assistant", "content": "Hello! How can I help?"}]
            }]
        }
    }]
)
```
{% endtab %}
{% tab title="TypeScript" %}
```typescript
await maniac.chat.completions.register({
  container: "my-container",
  items: [{
    input: {
      messages: [{ role: "user", content: "Hello!" }],
    },
    output: {
      choices: [{
        message: { role: "assistant", content: "Hello! How can I help?" },
      }],
    },
  }],
});
```
{% endtab %}
{% endtabs %}

{% embed url="https://youtu.be/5ygzMi4okJ8" %}

***

## Optimizing your model

The inference logs in your container now serve as training data for a new SLM—fully yours, lower latency, most cost effective, and optimized specifically for your task.

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

:e-mail: Email us at helpme@maniac.ai

We'll get back to you within a day.

[^1]: 
