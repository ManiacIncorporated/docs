# Quick Start Guide

Get up and running with Maniac in under 5 minutes.

## Prerequisites

- Python 3.9 or higher
- For Vertex AI: Google Cloud project with Anthropic models enabled
- For OpenAI: OpenAI API key

## Installation

Install the Maniac Python library using pip:

```bash
pip install maniac
```

## Basic Setup

### 1. Import and Initialize

```python
from maniac import Maniac

# For Vertex AI (recommended)
client = Maniac(
    provider="vertex",
    project_id="your-gcp-project-id",
    region="us-east5"
)

# For OpenAI
client = Maniac(
    provider="openai",
    api_key="your-openai-api-key"
)
```

### 2. Using the Chat Completions API

```python
# Standard chat completions interface
response = client.chat.completions.create(
    model="claude-opus-4",
    messages=[
        {"role": "system", "content": "You are a helpful math tutor."},
        {"role": "user", "content": "A train travels 120 miles in 2 hours. What is its average speed?"}
    ],
    temperature=0.0,
    task_label="math-problems",
    judge_prompt="Is the mathematical calculation correct and clearly explained?"
)

print(response["choices"][0]["message"]["content"])
# Output: "The average speed is 60 miles per hour. This is calculated by dividing distance (120 miles) by time (2 hours): 120 ÷ 2 = 60 mph."
```

### 3. Using the Responses API

```python
# Simplified responses interface
response = client.responses.create(
    model="claude-opus-4",
    input="A train travels 120 miles in 2 hours. What is its average speed?",
    instructions="You are a helpful math tutor. Solve the problem step by step.",
    temperature=0.0,
    max_tokens=1024,
    task_label="math-problems",
    judge_prompt="Is the mathematical calculation correct and clearly explained?"
)

print(response["output_text"])
# Output: "The average speed is 60 miles per hour..."
```

## Key Features

### Task Labeling
Group related inferences for optimization and tracking:

```python
# All inferences with the same task_label are grouped together
response1 = client.chat.completions.create(
    model="claude-opus-4",
    messages=[{"role": "user", "content": "Question 1"}],
    task_label="customer-support",
    judge_prompt="Is this response helpful and professional?"
)

response2 = client.chat.completions.create(
    model="claude-opus-4", 
    messages=[{"role": "user", "content": "Question 2"}],
    task_label="customer-support",  # Same task label
    judge_prompt="Is this response helpful and professional?"
)
```

### Available Models
- **Vertex AI**: `claude-opus-4`, `claude-opus-4.1`, `claude-sonnet-4`
- **OpenAI**: `gpt-4o`, `gpt-4`, `gpt-3.5-turbo`, `o1-mini`

### Batch Processing (Vertex AI only)
```python
from maniac import create_vertex_client

client = create_vertex_client(project_id="your-project")

# Create batch requests
requests = [
    {
        "messages": [{"role": "user", "content": "Question 1"}],
        "custom_id": "req_1"
    },
    {
        "messages": [{"role": "user", "content": "Question 2"}], 
        "custom_id": "req_2"
    }
]

# Submit batch job
job_name = client.provider.submit_batch_job(
    requests=requests,
    model="claude-opus-4",
    bucket_name="your-gcs-bucket"
)

# Check status
status = client.provider.get_batch_job_status(job_name)
print(f"Job state: {status['state']}")

# Get results when complete
if status['state'] == 'JOB_STATE_SUCCEEDED':
    results = client.provider.get_batch_results(job_name)
```

## Example: Building a Customer Support Agent

```python
from maniac import Maniac

client = Maniac(provider="vertex", project_id="your-project")

def handle_customer_query(query: str):
    response = client.responses.create(
        model="claude-opus-4",
        input=query,
        instructions="You are a helpful customer support agent. Be friendly, professional, and provide clear solutions.",
        temperature=0.0,
        max_tokens=1024,
        task_label="customer-support",
        judge_prompt="Is this response helpful, professional, and does it address the customer's question clearly?"
    )
    
    return response["output_text"]

# Use in your application
answer = handle_customer_query("How do I reset my password?")
print(answer)
```

## Troubleshooting

### Common Issues

**API Key Invalid**
```
ManiacAuthError: Invalid API key
```
Solution: Verify your API key in the [dashboard](https://dashboard.maniac.ai)

**Rate Limiting**
```
ManiacRateLimitError: Rate limit exceeded
```
Solution: Implement exponential backoff or upgrade your plan

**Model Not Available**
```
ManiacModelError: Requested model not available
```
Solution: The system will automatically fallback to the next best model

## Support

Need help? We're here for you:

- 📧 Email: support@maniac.ai
- 💬 Discord: [Join our community](https://discord.gg/maniac)
- 📚 Full docs: [documentation.maniac.ai](https://docs.maniac.ai)