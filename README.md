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
    judge_prompt="""
You are comparing two customer support ticket analyses for the same ticket. Is response A better than response B?

Consider these criteria:

ISSUE CATEGORIZATION:
- Does response A identify the primary issue type more correctly than B?
- Does response A identify secondary issues or complications better than B?
- Does response A use more appropriate support categories than B?

URGENCY ASSESSMENT:
- Does response A assess customer impact level more accurately than B?
- Does response A consider business/financial implications better than B?
- Does response A provide clearer urgency justification than B?

RESOLUTION PLAN:
- Does response A provide more specific, actionable next steps than B?
- Does response A include better timeline and ownership details than B?
- Does response A address customer communication needs better than B?
- Does response A consider escalation paths more appropriately than B?

Answer: A is better than B (YES/NO)
"""
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
    judge_prompt="""
You are comparing two legal document analyses for the same contract. Is response A better than response B?

Consider these criteria:

THOROUGHNESS:
- Does response A identify key contract terms and clauses more completely than B?
- Does response A review liability and risk provisions more thoroughly than B?
- Does response A examine termination and renewal terms better than B?

RISK IDENTIFICATION:
- Does response A highlight potential legal risks better than B?
- Does response A assess financial implications more accurately than B?
- Does response A identify compliance requirements more comprehensively than B?

ACTIONABILITY:
- Does response A provide clearer recommendations than B?
- Does response A suggest more specific negotiation points than B?

Answer: A is better than B (YES/NO)
"""
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
    judge_prompt="""
You are comparing two financial audit responses for the same expense report. Is response A better than response B?

Consider these criteria:

COMPLIANCE REVIEW:
- Does response A check adherence to company expense policies better than B?
- Does response A verify receipt documentation requirements more thoroughly than B?
- Does response A review approval workflows and authorization better than B?

ACCURACY & COMPLETENESS:
- Does response A validate expense calculations and totals more accurately than B?
- Does response A confirm expense categories more appropriately than B?
- Does response A identify missing information or documentation better than B?

RISK ASSESSMENT:
- Does response A identify potential fraud indicators better than B?
- Does response A flag unusual patterns or outliers more effectively than B?

Answer: A is better than B (YES/NO)
"""
)
```

**Responses API** (Simplified):

```python
response = client.responses.create(
    fallback="claude-opus-4",
    input="Expense report data...",
    instructions="You are a financial auditor. Review for compliance and accuracy.",
    task_label="expense-audit",
    judge_prompt="""
You are comparing two financial audit responses for the same expense report. Is response A better than response B?

Consider these criteria:

COMPLIANCE REVIEW:
- Does response A check adherence to company expense policies better than B?
- Does response A verify receipt documentation requirements more thoroughly than B?
- Does response A review approval workflows and authorization better than B?

ACCURACY & COMPLETENESS:
- Does response A validate expense calculations and totals more accurately than B?
- Does response A confirm expense categories more appropriately than B?
- Does response A identify missing information or documentation better than B?

RISK ASSESSMENT:
- Does response A identify potential fraud indicators better than B?
- Does response A flag unusual patterns or outliers more effectively than B?

Answer: A is better than B (YES/NO)
"""
)
```

#### Batch Processing

Process multiple requests efficiently (Vertex AI only):

```python
requests = [
    {"messages": [{"role": "user", "content": "Question 1"}], "custom_id": "req_1"},
    {"messages": [{"role": "user", "content": "Question 2"}], "custom_id": "req_2"}
]

job_name = client.provider.submit_batch_job(
    requests=requests,
    model="claude-opus-4",
    bucket_name="your-gcs-bucket"
)

# Check status and get results
status = client.provider.get_batch_job_status(job_name)
if status['state'] == 'JOB_STATE_SUCCEEDED':
    results = client.provider.get_batch_results(job_name)
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
* Email: support@maniac.ai
