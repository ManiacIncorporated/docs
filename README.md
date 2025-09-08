<p align="center">
  <img src="logo.png" alt="Maniac Logo" width="400"/>
</p>

# Maniac: LLM-Agnostic AI Program Orchestration

Maniac provides a unified interface for deploying AI programs across any LLM provider or model. Each inference creates an **AI Program Container** that continuously optimizes both prompts and LoRA fine-tuning parameters across all models, ensuring optimal performance regardless of which model the Control Plane allocates.

## What Maniac Does

Maniac is a Python library that provides:

- **Unified API**: Single interface for OpenAI and Vertex AI (Anthropic Claude) models
- **Telemetry & Tracking**: Automatic logging of all inferences for optimization
- **Task Organization**: Group related inferences with task labels
- **Quality Assessment**: Judge prompts for continuous evaluation
- **Batch Processing**: Efficient batch inference for Vertex AI models
- **Provider Abstraction**: Switch between providers without code changes

## Quick Start

```python
from maniac import Maniac

# Initialize with your preferred provider
client = Maniac(provider="vertex", project_id="your-project")
# or
client = Maniac(provider="openai", api_key="your-key")

# Customer support ticket analysis
response = client.responses.create(
    model="claude-opus-4",
    input="Customer reports: 'Payment failed but was charged anyway. Order #12345'", 
    instructions="You are a customer support analyst. Categorize the issue, determine urgency, and suggest resolution steps.",
    temperature=0.0,
    max_tokens=1024,
    task_label="support-ticket-analysis",
    judge_prompt="You are comparing two customer support analyses for the same ticket. Is response A's categorization and resolution plan at least as accurate and actionable as response B's?"
)

print(response["output_text"])
```

## Core Concepts

### AI Program Containers

Every inference line creates an AI Program Container that:

- **Continuously optimizes prompts and LoRA adaptations** across all models simultaneously
- **Maintains unified optimization state** combining prompt engineering and fine-tuning metrics
- **Handles seamless model switching** with pre-optimized prompts and LoRA weights
- **Automatically balances prompt vs LoRA optimization** based on model capabilities

### Task Labels & Judge Prompts

Maniac uses task labels to group related inferences and judge prompts to define quality criteria:

```python
# All inferences with the same task_label are grouped together
client.chat.completions.create(
    model="claude-opus-4",
    messages=[{"role": "user", "content": "Analyze this contract"}],
    task_label="legal-document-analysis",
    judge_prompt="Is the legal analysis thorough and does it identify key risks?"
)
```

### Supported Providers

- **Vertex AI (Recommended)**: Claude Opus 4, Claude Sonnet 4 via Google Cloud
- **OpenAI**: GPT-4o, GPT-4, GPT-3.5-turbo, O1-mini

## Key Features

### Two API Interfaces

**Chat Completions API** (OpenAI-compatible):
```python
response = client.chat.completions.create(
    model="claude-opus-4",
    messages=[
        {"role": "system", "content": "You are a financial auditor."},
        {"role": "user", "content": "Review this expense report..."}
    ],
    task_label="expense-audit",
    judge_prompt="Is the audit thorough and compliant?"
)
```

**Responses API** (Simplified):
```python
response = client.responses.create(
    model="claude-opus-4",
    input="Expense report data...",
    instructions="You are a financial auditor. Review for compliance and accuracy.",
    task_label="expense-audit",
    judge_prompt="Is the audit thorough and compliant?"
)
```

### Batch Processing

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

## Installation

```bash
pip install maniac
```

**For Vertex AI:** Requires Google Cloud project with Anthropic models enabled
**For OpenAI:** Requires OpenAI API key

## Enterprise Benefits

- **Vendor Independence**: Single API maintains operations across model providers
- **Automatic Optimization**: Continuous prompt and LoRA improvements using production data  
- **Quality Assurance**: Judge prompts ensure consistent output quality
- **Cost Management**: Efficient batch processing and provider flexibility
- **Complete Audit Trail**: All inferences logged for compliance and optimization

## Documentation

- [Quick Start Guide](getting-started/quickstart.md)
- [Python SDK Reference](api/python-sdk.md)
- [Installation & Authentication](getting-started/installation.md)
- [Best Practices](guides/best-practices.md)

## Support

For issues and questions:
- GitHub Issues: [github.com/maniaclabs/maniac](https://github.com/maniaclabs/maniac)
- Documentation: [docs.maniac.ai](https://docs.maniac.ai)
- Email: support@maniac.ai
