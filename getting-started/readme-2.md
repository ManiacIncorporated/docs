---
hidden: true
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

# Copy of Maniac: Your best model in one click.

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

#### Install the library

```python
pip install maniac
```

#### Initialize client

<pre class="language-python"><code class="lang-python"><a data-footnote-ref href="#user-content-fn-1">from maniac import Maniac</a>

# Simple initialization - Maniac handles all providers automatically
maniac = Maniac(api_key="your-maniac-api-key")
</code></pre>

#### Create a container

Containers log inference and automatically build datasets for fine-tuning and evaluation. <mark style="color:red;">`initial_model`</mark> sets the model used in that container until a Maniac model is deployed in its place.

```python
container = maniac.containers.create(
  label = "my-container"
  initial_model = "openai/gpt-5",
  default_system_prompt = "You are a helpful math tutor."
)
```

#### Log Completions

Now that you've made a container, let's add some data to it. Maniac mirrors your completions with any inference provider to auto-populate inference logs in your container. Your inference cost and latency are unaffected. External datasets can also be [manually uploaded](../datasets/upload-datasets.md).&#x20;

{% tabs %}
{% tab title="Chat Completions API" %}
```python
response = maniac.chat.completions.register(
    model = "maniac:my-container",
    messages = [{"role": "user", "content": "A train travels 120 miles in 2 hours. What is its average speed?"}],
    judge_prompt = "Compare two math solutions. Is A better than B? Consider: calculation accuracy, clear explanations, educational value."
    reasoning = {"effort": "medium"} # Optional reasoning parameter
)

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
<div data-with-frame="true"><figure><img src="../.gitbook/assets/Screenshot 2026-01-06 at 12.43.14 PM.png" alt=""><figcaption></figcaption></figure></div>
{% endtab %}

{% tab title="Code Eval" %}
<div data-with-frame="true"><figure><img src="../.gitbook/assets/Screenshot 2026-01-06 at 1.15.10 PM.png" alt=""><figcaption></figcaption></figure></div>


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

<div data-with-frame="true"><figure><img src="../.gitbook/assets/Screenshot 2026-01-06 at 2.05.29 PM.png" alt=""><figcaption></figcaption></figure></div>

### Need help?

:e-mail: Email us at helpme@maniac.ai

We'll get back to you within a day.

[^1]: 
