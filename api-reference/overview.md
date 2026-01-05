---
layout:
  width: default
  title:
    visible: false
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

# Overview

## API Reference

The Maniac API provides openai compatible inference endpoints for interacting with both frontier models and your custom models. This reference details the available endpoints.&#x20;

## Create a new user

<mark style="color:green;">`POST`</mark> `/users`

\<Description of the endpoint>

**Headers**

| Name          | Value              |
| ------------- | ------------------ |
| Content-Type  | `application/json` |
| Authorization | `Bearer <token>`   |

**Body**

| Name   | Type   | Description      |
| ------ | ------ | ---------------- |
| `name` | string | Name of the user |
| `age`  | number | Age of the user  |

**Response**

{% tabs %}
{% tab title="200" %}
```json
{
  "id": 1,
  "name": "John",
  "age": 30
}
```
{% endtab %}

{% tab title="400" %}
```json
{
  "error": "Invalid request"
}
```
{% endtab %}
{% endtabs %}



## Client Initialization

### Maniac Class

```python
from maniac import Maniac

client = Maniac(
    api_key="your-maniac-api-key",  # Your Maniac API key
    base_url=None                   # Optional custom endpoint
)
```

**Parameters:**

* `api_key` (str): Your Maniac API key (handles all provider routing)
* `base_url` (str): Optional custom API endpoint (for enterprise deployments)

### Factory Functions (Legacy - use Maniac() instead)

```python
from maniac import Maniac

# Recommended: Direct initialization
client = Maniac(api_key="your-maniac-api-key")

# Legacy factory functions (deprecated)
from maniac import create_client
client = create_client(api_key="your-maniac-api-key")
```

## Chat Completions API

### client.chat.completions.create()

Standard OpenAI-compatible chat completions interface.

```python
response = client.chat.completions.create(
    fallback="claude-opus-4",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Hello!"}
    ],
    temperature=0.7,
    max_tokens=1024,
    top_p=1.0,
    stream=False,
    task_label="conversation",                   # Required
    judge_prompt="Compare two conversational responses. Is A better than B? Consider: helpfulness, appropriate tone, engagement."      # Required
)
```

**Required Parameters:**

* `fallback` (str): Model name (e.g., "claude-opus-4", "gpt-4o")
* `messages` (List\[Dict]): Chat messages in OpenAI format
* `task_label` (str): Task identifier for grouping and optimization
* `judge_prompt` (str): Evaluation criteria for quality assessment

**Optional Parameters:**

* `temperature` (float): Randomness (0.0-2.0)
* `max_tokens` (int): Maximum response tokens
* `top_p` (float): Nucleus sampling parameter
* `stream` (bool): Enable streaming responses
* `system` (str): System message (alternative to system role in messages)

**Response Format:**

```python
{
    "id": "completion-id",
    "object": "chat.completion", 
    "created": 1234567890,
    "model": "claude-opus-4",
    "choices": [{
        "index": 0,
        "message": {
            "role": "assistant",
            "content": "Hello! How can I help you today?"
        },
        "finish_reason": "stop"
    }],
    "usage": {
        "prompt_tokens": 10,
        "completion_tokens": 15,
        "total_tokens": 25
    }
}
```

### client.chat.completions.stream\_create()

Log pre-generated responses without making API calls.

```python
client.chat.completions.stream_create(
    task_label="conversation",
    system_prompt="You are a helpful assistant.", 
    user_prompt="Hello!",
    output="Hello! How can I help you today?",
    judge_prompt="Compare two conversational responses. Is A better than B? Consider: helpfulness, appropriate tone, engagement.",
    model_name="claude-opus-4"
)
```

## Responses API

### client.responses.create()

Simplified interface using input/instructions pattern.

```python
response = client.responses.create(
    fallback="claude-opus-4",
    input="Analyze this customer feedback",
    instructions="You are a customer success manager. Categorize the feedback and suggest actions.",
    additional_instructions="Focus on actionable insights.",  # Optional
    temperature=0.0,
    max_tokens=2048,
    task_label="feedback-analysis",          # Required  
    judge_prompt="Compare two feedback analyses. Is A better than B? Consider: insight depth, actionable recommendations, business relevance."  # Required
)
```

**Required Parameters:**

* `fallback` (str): Model name (e.g., "claude-opus-4", "gpt-4o")
* `input` (str): The content to process
* `task_label` (str): Task identifier
* `judge_prompt` (str): Evaluation criteria

**Optional Parameters:**

* `instructions` (str): System instructions/role definition
* `additional_instructions` (str): Additional context or constraints
* `temperature` (float): Randomness (0.0-2.0)
* `max_tokens` (int): Maximum response tokens
* `top_p` (float): Nucleus sampling parameter
* `stream` (bool): Enable streaming
* `tools` (List\[Dict]): Tool definitions

**Response Format:**

```python
{
    "id": "response-id",
    "object": "responses.response",
    "created": 1234567890,
    "model": "claude-opus-4",
    "output_text": "Based on the feedback...",
    "choices": [/* OpenAI-compatible format */],
    "usage": {
        "prompt_tokens": 20,
        "completion_tokens": 100,
        "total_tokens": 120
    }
}
```

## Supported Models

Maniac automatically routes to the optimal provider for each model:

### Claude Models (Anthropic via Vertex AI)

* `claude-opus-4` - Most capable model for complex reasoning
* `claude-sonnet-4` - Balanced performance and speed
* `claude-haiku-3` - Fast model for simple tasks

### GPT Models (OpenAI)

* `gpt-4o` - Latest multimodal model
* `gpt-4-turbo` - High performance model
* `gpt-4` - Classic high-quality model
* `gpt-3.5-turbo` - Fast and cost-effective
* `o1-mini` - Reasoning-optimized model

### Gemini Models (Google)

* `gemini-pro` - Google's advanced model
* `gemini-1.5-pro` - Latest version with long context

### Open Source Models

* `llama-3.1-70b` - Meta's large language model
* `mixtral-8x7b` - Mistral's mixture-of-experts model
* `codestral` - Specialized for code generation

## Batch Processing

### Submit Batch Job

```python
requests = [
    {
        "fallback": "claude-opus-4",
        "messages": [{"role": "user", "content": "Question 1"}],
        "task_label": "batch-analysis",
        "judge_prompt": "Is this response accurate and helpful?",
        "max_tokens": 1024,
        "temperature": 0.0
    },
    {
        "fallback": "claude-opus-4",
        "messages": [{"role": "user", "content": "Question 2"}],
        "task_label": "batch-analysis", 
        "judge_prompt": "Is this response accurate and helpful?",
        "max_tokens": 1024
    }
]

# Maniac handles provider routing automatically
batch_id = client.submit_batch(requests=requests)
```

### Check Job Status

```python
status = client.get_batch_status(batch_id)

print(status)
# {
#     "batch_id": "batch_123456",
#     "state": "processing",
#     "created_at": "2024-01-01T12:00:00Z",
#     "updated_at": "2024-01-01T12:05:00Z",
#     "completed": 150,
#     "total": 200,
#     "failed": 2
# }
```

### Get Batch Results

```python
if status['state'] == 'completed':
    results = client.get_batch_results(batch_id)
    
    for result in results:
        print(f"Model used: {result['model_used']}")
        print(f"Response: {result['response']}")
        print(f"Provider: {result['provider_used']}")
```

## Model Management

### Get Available Models

```python
# Get all available models
models = client.get_available_models()
print(models)

# Get models by category
claude_models = client.get_available_models(category="claude")
gpt_models = client.get_available_models(category="gpt")
open_source = client.get_available_models(category="open-source")
```

### Model Information

```python
# Get model details
info = client.get_model_info("claude-opus-4")
print(f"Provider: {info['provider']}")
print(f"Context length: {info['max_context']}")
print(f"Capabilities: {info['capabilities']}")
```

## Telemetry and Logging

All API calls are automatically logged to Supabase for optimization and tracking:

* **Task grouping**: Inferences with same `task_label` are grouped
* **Performance tracking**: Latency and token usage recorded
* **Quality evaluation**: `judge_prompt` used for continuous improvement
* **Model routing**: System learns optimal model selection over time

## Error Handling

```python
from maniac import Maniac

try:
    client = Maniac(api_key="invalid-api-key")
    response = client.chat.completions.create(
        fallback="claude-opus-4",
        messages=[{"role": "user", "content": "Hello"}],
        task_label="test",
        judge_prompt="Compare two responses. Is A better than B? Consider: appropriateness, accuracy, clarity."
    )
except ValueError as e:
    print(f"Configuration error: {e}")
except ImportError as e:
    print(f"Missing dependencies: {e}")
except Exception as e:
    print(f"API error: {e}")
```

## Best Practices

### 1. Task Labeling

Use descriptive task labels to group related inferences:

```python
# Good
task_label="customer-support-password-reset"
task_label="document-summarization-legal" 
task_label="code-review-python"

# Bad  
task_label="test"
task_label="query"
```

### 2. Judge Prompts

Write specific evaluation criteria:

```python
# Good
judge_prompt="Compare two code reviews. Is A better than B? Consider: thoroughness, actionable suggestions, clarity."

# Bad
judge_prompt="Is this good?"
```

### 3. Temperature Settings

* Use `temperature=0.0` for deterministic tasks (analysis, classification)
* Use `temperature=0.7-1.0` for creative tasks (writing, brainstorming)

### 4. Provider Selection

* **Vertex AI**: Recommended for production use with Claude models
* **OpenAI**: Use for GPT model access or specific requirements

## Environment Variables

Set these environment variables for easier configuration:

```bash
# Set your Maniac API key
export MANIAC_API_KEY=your-maniac-api-key
```

Then initialize:

```python
import os
from maniac import Maniac

client = Maniac(api_key=os.getenv("MANIAC_API_KEY"))
```

