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

# Maniac: LLM-Agnostic AI Program Orchestration

Maniac provides a unified interface for deploying AI programs across any LLM provider or model. Each inference creates an **AI Program Container** that continuously optimizes both prompts and LoRA fine-tuning parameters across all models, ensuring optimal performance regardless of which model the Control Plane allocates.

### What Maniac Does

Maniac is a Python library that provides:

* **Model-Agnostic Approach**: Deploy on task-specific weights and prompts for any model
* **Intelligent Routing**: Automatic model selection and failover handling
* **Telemetry & Tracking**: Automatic logging of all inferences for optimization
* **Task Organization**: Group related inferences with task labels
* **Quality Assessment**: Judge prompts for continuous, automated evaluation and tuning
* **Provider Agnostic**: Access any model without worrying about underlying provider

### Quick Start

```python
from maniac import Maniac

# Initialize with your API key - Maniac handles all providers automatically
client = Maniac(api_key="your-maniac-api-key")

# Customer support ticket analysis
response = client.responses.create(
    fallback="claude-opus-4",
    input="Customer reports: 'Payment failed but was charged anyway. Order #12345'", 
    instructions="You are a customer support analyst. Categorize the issue, determine urgency, and suggest resolution steps.",
    temperature=0.0,
    max_tokens=1024,
    task_label="support-ticket-analysis",
    judge_prompt="Compare two customer support analyses. Is A better than B? Consider: issue identification, urgency assessment, actionable solutions."
)

print(response["output_text"])
```

### Core Concepts

#### AI Program Containers

Every inference line creates an AI Program Container that:

* **Continuously optimizes prompts and LoRA adaptations** across all models simultaneously
* **Maintains unified optimization state** combining prompt engineering and fine-tuning metrics
* **Handles seamless model switching** with pre-optimized prompts and LoRA weights
* **Automatically balances prompt vs LoRA optimization** based on model capabilities

#### Task Labels & Judge Prompts

Maniac uses task labels to group related inferences and judge prompts to define quality criteria:

```python
# All inferences with the same task_label are grouped together
client.chat.completions.create(
    fallback="claude-opus-4",
    messages=[{"role": "user", "content": "Analyze this contract"}],
    task_label="legal-document-analysis",
    judge_prompt="Compare two legal contract analyses. Is A better than B? Consider: completeness, risk identification, actionable recommendations."
)
```

#### Supported Models

Maniac automatically routes to the optimal provider for each model:

* **Claude Models**: claude-opus-4, claude-sonnet-4, claude-haiku-3
* **GPT Models**: gpt-4o, gpt-4-turbo, gpt-4, gpt-3.5-turbo, o1-mini
* **Gemini Models**: gemini-pro, gemini-1.5-pro
* **Open Source**: llama-3.1-70b, mixtral-8x7b, codestral

### Key Features

#### Two API Interfaces

**Chat Completions API** (OpenAI-compatible):

```python
response = client.chat.completions.create(
    fallback="claude-opus-4",
    messages=[
        {"role": "system", "content": "You are a financial auditor."},
        {"role": "user", "content": "Review this expense report..."}
    ],
    task_label="expense-audit",
    judge_prompt="Compare two financial audit reviews. Is A better than B? Consider: policy compliance, calculation accuracy, fraud detection."
)
```

**Responses API** (Simplified):

```python
response = client.responses.create(
    fallback="claude-opus-4",
    input="Expense report data...",
    instructions="You are a financial auditor. Review for compliance and accuracy.",
    task_label="expense-audit",
    judge_prompt="Compare two financial audit reviews. Is A better than B? Consider: policy compliance, calculation accuracy, fraud detection."
)
```

#### Batch Processing

Process multiple requests efficiently:

```python
requests = [
    {
        "fallback": "claude-opus-4",
        "messages": [{"role": "user", "content": "Question 1"}],
        "task_label": "batch-analysis",
        "judge_prompt": "Compare two responses. Is A better than B?"
    },
    {
        "fallback": "claude-opus-4",
        "messages": [{"role": "user", "content": "Question 2"}],
        "task_label": "batch-analysis",
        "judge_prompt": "Compare two responses. Is A better than B?"
    }
]

# Submit batch job
batch_id = client.submit_batch(requests=requests)

# Check status and get results
status = client.get_batch_status(batch_id)
if status['state'] == 'completed':
    results = client.get_batch_results(batch_id)
```

### Installation

```bash
pip install maniac
```

**Prerequisites:** Just your Maniac API key - all model providers are handled automatically

### Enterprise Benefits

* **Vendor Independence**: Single API maintains operations across model providers
* **Automatic Optimization**: Continuous prompt and LoRA improvements using production data
* **Quality Assurance**: Judge prompts ensure consistent output quality
* **Cost Management**: Efficient batch processing and provider flexibility
* **Complete Audit Trail**: All inferences logged for compliance and optimization

### Documentation

* [Quick Start Guide](getting-started/quickstart.md)
* [Python SDK Reference](api/python-sdk.md)
* [Installation & Authentication](getting-started/installation.md)
* [Best Practices](guides/best-practices.md)

### Support

For issues and questions:

* GitHub Issues: [github.com/maniaclabs/maniac](https://github.com/maniaclabs/maniac)
* Documentation: [docs.maniac.ai](https://docs.maniac.ai)
* Email: dhruv@maniaclabs.xyz or support@maniac.ai
