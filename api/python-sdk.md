# Python SDK Reference

Complete API reference for the Maniac Python library.

## Installation

```bash
pip install maniac
```

## Client Initialization

### Maniac Class

```python
from maniac import Maniac

client = Maniac(
    provider="vertex",           # "vertex" or "openai" 
    project_id="your-project",   # Required for Vertex AI
    region="us-east5",          # Optional, default "us-east5"
    api_key="your-key",         # Required for OpenAI
    base_url=None              # Optional OpenAI base URL
)
```

**Parameters:**
- `provider` (str): AI provider - "vertex" (recommended) or "openai"
- `project_id` (str): Google Cloud project ID (required for Vertex AI)
- `region` (str): Google Cloud region (optional, default: "us-east5")
- `api_key` (str): API key (required for OpenAI)
- `base_url` (str): Base URL for OpenAI API (optional)

### Factory Functions

```python
from maniac import create_vertex_client, create_openai_client, create_client

# Vertex AI client
client = create_vertex_client(
    project_id="your-project",
    region="us-east5"
)

# OpenAI client  
client = create_openai_client(
    api_key="your-key",
    base_url="https://api.openai.com/v1"  # optional
)

# Generic client
client = create_client(
    provider="vertex",
    project_id="your-project"
)
```

## Chat Completions API

### client.chat.completions.create()

Standard OpenAI-compatible chat completions interface.

```python
response = client.chat.completions.create(
    model="claude-opus-4",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Hello!"}
    ],
    temperature=0.7,
    max_tokens=1024,
    top_p=1.0,
    stream=False,
    task_label="conversation",                    # Required
    judge_prompt="""
You are comparing two conversational responses to the same user query. Is response A better than response B?

Consider these criteria:

HELPFULNESS:
- Does response A address the user's needs more appropriately than B?
- Does response A provide more useful information or assistance than B?
- Does response A encourage further interaction better than B when appropriate?

APPROPRIATENESS:
- Does response A's tone match the context and user's style better than B's?
- Is response A's length more suitable for the query than B's?
- Does response A maintain professional boundaries better than B?

ENGAGEMENT:
- Is response A more interesting and engaging than B?
- Does response A show better understanding of user intent than B?

Answer: A is better than B (YES/NO)
"""      # Required
)
```

**Required Parameters:**
- `model` (str): Model name
- `messages` (List[Dict]): Chat messages in OpenAI format
- `task_label` (str): Task identifier for grouping and optimization
- `judge_prompt` (str): Evaluation criteria for quality assessment

**Optional Parameters:**
- `temperature` (float): Randomness (0.0-2.0)
- `max_tokens` (int): Maximum response tokens
- `top_p` (float): Nucleus sampling parameter
- `stream` (bool): Enable streaming responses
- `system` (str): System message (alternative to system role in messages)

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

### client.chat.completions.stream_create()

Log pre-generated responses without making API calls.

```python
client.chat.completions.stream_create(
    task_label="conversation",
    system_prompt="You are a helpful assistant.", 
    user_prompt="Hello!",
    output="Hello! How can I help you today?",
    judge_prompt="""
You are comparing two conversational responses to the same user query. Is response A better than response B?

Consider these criteria:

HELPFULNESS:
- Does response A address the user's needs more appropriately than B?
- Does response A provide more useful information or assistance than B?
- Does response A encourage further interaction better than B when appropriate?

APPROPRIATENESS:
- Does response A's tone match the context and user's style better than B's?
- Is response A's length more suitable for the query than B's?
- Does response A maintain professional boundaries better than B?

ENGAGEMENT:
- Is response A more interesting and engaging than B?
- Does response A show better understanding of user intent than B?

Answer: A is better than B (YES/NO)
""",
    model_name="claude-opus-4"
)
```

## Responses API

### client.responses.create()

Simplified interface using input/instructions pattern.

```python
response = client.responses.create(
    model="claude-opus-4",
    input="Analyze this customer feedback",
    instructions="You are a customer success manager. Categorize the feedback and suggest actions.",
    additional_instructions="Focus on actionable insights.",  # Optional
    temperature=0.0,
    max_tokens=2048,
    task_label="feedback-analysis",           # Required  
    judge_prompt="""
You are comparing two feedback analysis responses for the same customer feedback dataset. Is response A better than response B?

Consider these criteria:

ANALYTICAL DEPTH:
- Does response A identify key themes and patterns better than B?
- Does response A categorize feedback more appropriately than B?
- Does response A draw more meaningful insights from the data than B?

ACTIONABILITY:
- Does response A provide more specific, implementable recommendations than B?
- Does response A prioritize actions by impact/effort better than B?
- Does response A include better timeline or next steps than B?

BUSINESS RELEVANCE:
- Does response A align with business objectives better than B?
- Does response A consider resource constraints and feasibility better than B?
- Does response A address customer satisfaction impact better than B?

Answer: A is better than B (YES/NO)
"""  # Required
)
```

**Required Parameters:**
- `model` (str): Model name
- `input` (str): The content to process
- `task_label` (str): Task identifier 
- `judge_prompt` (str): Evaluation criteria

**Optional Parameters:**
- `instructions` (str): System instructions/role definition
- `additional_instructions` (str): Additional context or constraints
- `temperature` (float): Randomness (0.0-2.0)
- `max_tokens` (int): Maximum response tokens  
- `top_p` (float): Nucleus sampling parameter
- `stream` (bool): Enable streaming
- `tools` (List[Dict]): Tool definitions

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

### Vertex AI Models
- `claude-opus-4` → `claude-opus-4@20250514`
- `claude-opus-4.1` → `claude-opus-4-1@20250805`  
- `claude-sonnet-4` → `claude-sonnet-4@20250514`

### OpenAI Models
- `gpt-4o`
- `gpt-4-turbo`
- `gpt-4`
- `gpt-3.5-turbo`
- `o1-mini`

## Batch Processing (Vertex AI Only)

### Submit Batch Job

```python
requests = [
    {
        "messages": [{"role": "user", "content": "Question 1"}],
        "custom_id": "req_1",
        "max_tokens": 1024,
        "temperature": 0.0
    },
    {
        "messages": [{"role": "user", "content": "Question 2"}],
        "custom_id": "req_2"
    }
]

job_name = client.provider.submit_batch_job(
    requests=requests,
    model="claude-opus-4", 
    bucket_name="your-gcs-bucket",
    input_file_prefix="batch_input",    # Optional
    output_file_prefix="batch_output",  # Optional
    max_tokens=4096,                   # Common parameters
    temperature=0.0
)
```

### Check Job Status

```python
status = client.provider.get_batch_job_status(job_name)

print(status)
# {
#     "name": "projects/.../batchPredictionJobs/123",
#     "display_name": "batch_prediction_claude-opus-4_1234567890", 
#     "state": "JOB_STATE_RUNNING",
#     "create_time": "2024-01-01T12:00:00Z",
#     "update_time": "2024-01-01T12:05:00Z",
#     "error": None
# }
```

### Get Batch Results

```python
if status['state'] == 'JOB_STATE_SUCCEEDED':
    results = client.provider.get_batch_results(job_name)
    
    for result in results:
        print(f"Request: {result['custom_id']}")
        print(f"Response: {result['response']}")
```

## Provider Access

### Direct Provider Methods

```python
# Access underlying provider for advanced features
vertex_provider = client.provider

# Get available models
models = vertex_provider.get_available_models()
print(models)  # ['claude-opus-4', 'claude-opus-4.1', 'claude-sonnet-4']

# Create JSONL for batch processing
jsonl_content = vertex_provider.create_batch_jsonl(
    requests=batch_requests,
    model="claude-opus-4",
    temperature=0.0
)
```

## Telemetry and Logging

All API calls are automatically logged to Supabase for optimization and tracking:

- **Task grouping**: Inferences with same `task_label` are grouped
- **Performance tracking**: Latency and token usage recorded
- **Quality evaluation**: `judge_prompt` used for continuous improvement
- **Model routing**: System learns optimal model selection over time

## Error Handling

```python
from maniac import Maniac

try:
    client = Maniac(provider="vertex", project_id="invalid-project")
    response = client.chat.completions.create(
        model="claude-opus-4",
        messages=[{"role": "user", "content": "Hello"}],
        task_label="test",
        judge_prompt="""
You are comparing two responses to the same input. Is response A better than response B?

Consider these criteria:

APPROPRIATENESS:
- Does response A address the input more appropriately than B?
- Is response A's tone more suitable for the context than B's?
- Is response A's length more appropriate than B's?

QUALITY:
- Is response A's information more accurate and helpful than B's?
- Is response A's language clearer and more understandable than B's?

Answer: A is better than B (YES/NO)
"""
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
judge_prompt="""
You are comparing two code reviews for the same piece of code. Is response A better than response B?

Consider these criteria:

THOROUGHNESS:
- Does response A identify more potential bugs and issues than B?
- Does response A review code structure and organization better than B?
- Does response A check for security vulnerabilities more thoroughly than B?
- Does response A validate error handling better than B?

IMPROVEMENT SUGGESTIONS:
- Does response A provide more specific, actionable recommendations than B?
- Does response A suggest better performance optimizations than B?
- Does response A recommend more appropriate best practices than B?

CLARITY & PROFESSIONALISM:
- Are response A's explanations clearer and more constructive than B's?
- Does response A maintain a more respectful tone than B?

Answer: A is better than B (YES/NO)
"""

# Bad
judge_prompt="Is this good?"
```

### 3. Temperature Settings
- Use `temperature=0.0` for deterministic tasks (analysis, classification)
- Use `temperature=0.7-1.0` for creative tasks (writing, brainstorming)

### 4. Provider Selection
- **Vertex AI**: Recommended for production use with Claude models
- **OpenAI**: Use for GPT model access or specific requirements

## Environment Variables

Set these environment variables for easier configuration:

```bash
# For Vertex AI
export GOOGLE_CLOUD_PROJECT=your-project-id
export GOOGLE_CLOUD_REGION=us-east5

# For OpenAI  
export OPENAI_API_KEY=your-api-key
export OPENAI_BASE_URL=https://api.openai.com/v1
```

Then initialize without parameters:

```python
import os
from maniac import Maniac

client = Maniac(
    provider="vertex",
    project_id=os.getenv("GOOGLE_CLOUD_PROJECT"),
    region=os.getenv("GOOGLE_CLOUD_REGION")
)
```