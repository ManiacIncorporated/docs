# Provider Selection

Maniac provides a unified interface across multiple AI providers, allowing you to switch between OpenAI and Vertex AI without changing your application code.

## Supported Providers

### Vertex AI (Recommended)

Access Anthropic Claude models through Google Cloud:

```python
from maniac import Maniac

client = Maniac(
    provider="vertex",
    project_id="your-gcp-project", 
    region="us-east5"  # Optional, default is us-east5
)
```

**Available Models:**
- `claude-opus-4` → `claude-opus-4@20250514`
- `claude-opus-4.1` → `claude-opus-4-1@20250805`  
- `claude-sonnet-4` → `claude-sonnet-4@20250514`

**Features:**
- Batch processing support
- Google Cloud integration
- Enterprise-grade reliability
- Advanced model versions

### OpenAI

Access GPT models through OpenAI API:

```python
from maniac import Maniac

client = Maniac(
    provider="openai",
    api_key="your-maniac-api-key",
    base_url="https://api.openai.com/v1"  # Optional
)
```

**Available Models:**
- `gpt-4o`
- `gpt-4-turbo`
- `gpt-4`
- `gpt-3.5-turbo`
- `o1-mini`

**Features:**
- Direct OpenAI API access
- Full GPT model support
- Custom base URL support

## Provider Switching

The same code works across both providers:

```python
# This works with both Vertex AI and OpenAI
response = client.chat.completions.create(
    model="claude-opus-4",  # or "gpt-4o" 
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Explain quantum computing"}
    ],
    temperature=0.7,
    max_tokens=1024,
    task_label="educational-content",
    judge_prompt="""
You are comparing two educational explanations of the same topic. Is response A better than response B?

Consider these criteria:

CLARITY:
- Does response A use language more appropriate for the target audience than B?
- Does response A explain complex concepts in more understandable terms than B?
- Does response A have better logical flow and structure than B?

ACCURACY:
- Is response A's scientific/technical information more correct than B's?
- Does response A have fewer misleading or false statements than B?

COMPLETENESS:
- Does response A cover key concepts more adequately than B?
- Does response A provide more relevant examples or analogies than B?

Answer: A is better than B (YES/NO)
"""
)
```

## Factory Functions

Use convenience functions for cleaner initialization:

### Vertex AI Client

```python
from maniac import create_vertex_client

client = create_vertex_client(
    project_id="your-project",
    region="us-east5"
)
```

### OpenAI Client

```python
from maniac import create_openai_client

client = create_openai_client(
    api_key="your-maniac-api-key",
    base_url="https://api.openai.com/v1"  # Optional
)
```

### Generic Client

```python
from maniac import create_client

# Vertex AI
vertex_client = create_client(
    provider="vertex",
    project_id="your-project"
)

# OpenAI  
openai_client = create_client(
    provider="openai",
    api_key="your-maniac-api-key"
)
```

## Provider-Specific Features

### Vertex AI Batch Processing

Only available with Vertex AI provider:

```python
vertex_client = create_vertex_client(project_id="your-project")

# Submit batch job
job_name = vertex_client.provider.submit_batch_job(
    requests=[
        {"messages": [{"role": "user", "content": "Question 1"}], "custom_id": "req_1"},
        {"messages": [{"role": "user", "content": "Question 2"}], "custom_id": "req_2"}
    ],
    model="claude-opus-4",
    bucket_name="your-gcs-bucket"
)

# Check status
status = vertex_client.provider.get_batch_job_status(job_name)

# Get results when complete
if status['state'] == 'JOB_STATE_SUCCEEDED':
    results = vertex_client.provider.get_batch_results(job_name)
```

### Model Information

Get provider-specific model information:

```python
# Get available models for current provider
if client.provider_type == "vertex":
    models = client.provider.get_available_models()
    print(models)  # ['claude-opus-4', 'claude-opus-4.1', 'claude-sonnet-4']
elif client.provider_type == "openai":
    # OpenAI models are accessed directly
    models = ["gpt-4o", "gpt-4-turbo", "gpt-4", "gpt-3.5-turbo", "o1-mini"]
```

## Environment Configuration

Set environment variables for easier configuration:

### Vertex AI

```bash
export GOOGLE_CLOUD_PROJECT=your-project-id
export GOOGLE_CLOUD_REGION=us-east5
```

### OpenAI

```bash
export MANIAC_API_KEY=your-maniac-api-key
```

Then initialize without parameters:

```python
import os
from maniac import Maniac

# Vertex AI
client = Maniac(
    provider="vertex",
    project_id=os.getenv("GOOGLE_CLOUD_PROJECT"),
    region=os.getenv("GOOGLE_CLOUD_REGION")
)

# OpenAI
client = Maniac(
    provider="openai", 
    api_key=os.getenv("MANIAC_API_KEY")
)
```

## Choosing the Right Provider

### Use Vertex AI When:
- You need Claude models (Opus 4, Sonnet 4)
- You want batch processing capabilities
- You're already using Google Cloud
- You need enterprise-grade compliance
- You want the latest Anthropic model versions

### Use OpenAI When:
- You need GPT models specifically
- You prefer the OpenAI model ecosystem
- You need O1-series models

*Note: OpenAI access is provisioned automatically through your Maniac API key*

## Provider Migration

Migrating between providers requires only changing the initialization:

```python
# Before (OpenAI)
client = Maniac(provider="openai", api_key="your-maniac-api-key")

# After (Vertex AI)  
client = Maniac(provider="vertex", project_id="your-project")

# All other code remains the same
response = client.chat.completions.create(
    model="claude-opus-4",  # Just change the model name
    messages=messages,
    task_label="same-task-label",
    judge_prompt="same-judge-prompt"
)
```

## Best Practices

### 1. Provider Selection
Choose based on your model and infrastructure requirements, not routing complexity.

### 2. Model Names
Use the correct model names for each provider:
- Vertex AI: `claude-opus-4`, `claude-sonnet-4`  
- OpenAI: `gpt-4o`, `gpt-4-turbo`

### 3. Task Labels
Keep task labels consistent across providers for better optimization tracking.

### 4. Error Handling
Handle provider-specific errors appropriately:

```python
try:
    response = client.chat.completions.create(...)
except ImportError as e:
    print(f"Missing dependencies for {client.provider_type}: {e}")
except ValueError as e:
    print(f"Configuration error: {e}")
```

### 5. Authentication
Ensure proper authentication is set up for your chosen provider before initialization.

## Next Steps

- [Learn about API interfaces](../api/python-sdk.md)
- [Set up authentication](../getting-started/authentication.md) 
- [Explore batch processing](../guides/batch-processing.md)