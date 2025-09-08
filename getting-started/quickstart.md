# Quick Start Guide

Get up and running with Maniac in under 5 minutes.

## Prerequisites

- Python 3.9 or higher
- For Vertex AI: Google Cloud project with Anthropic models enabled
- For OpenAI: Access provisioned automatically via Maniac API key

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
    api_key="your-maniac-api-key"
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
    judge_prompt="""
You are comparing two math tutoring responses for the same problem. Is response A better than response B?

Consider these criteria:

ACCURACY:
- Is response A's calculation more mathematically correct than B's?
- Does response A handle units better than response B?
- Is response A's formula application more accurate than B's?

EXPLANATION CLARITY:
- Does response A show a clearer step-by-step process than B?
- Does response A explain reasoning better than response B?
- Does response A use more appropriate mathematical terminology than B?

EDUCATIONAL VALUE:
- Does response A help the student understand better than B?
- Does response A provide better context or real-world connections than B?

Answer: A is better than B (YES/NO)
"""
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
    judge_prompt="""
You are comparing two math tutoring responses for the same problem. Is response A better than response B?

Consider these criteria:

ACCURACY:
- Is response A's calculation more mathematically correct than B's?
- Does response A handle units better than response B?
- Is response A's formula application more accurate than B's?

EXPLANATION CLARITY:
- Does response A show a clearer step-by-step process than B?
- Does response A explain reasoning better than response B?
- Does response A use more appropriate mathematical terminology than B?

EDUCATIONAL VALUE:
- Does response A help the student understand better than B?
- Does response A provide better context or real-world connections than B?

Answer: A is better than B (YES/NO)
"""
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
    judge_prompt="""
You are comparing two customer support responses to the same customer inquiry. Is response A better than response B?

Consider these criteria:

HELPFULNESS:
- Does response A address the customer's question more directly than B?
- Does response A provide more actionable next steps or solutions than B?
- Does response A anticipate follow-up questions better than B?

PROFESSIONALISM:
- Does response A use more appropriate tone and language than B?
- Does response A show more empathy and understanding than B?
- Does response A maintain company brand voice better than B?

ACCURACY:
- Is response A's information more factually correct than B's?
- Are response A's policy/procedure references more accurate than B's?

Answer: A is better than B (YES/NO)
"""
)

response2 = client.chat.completions.create(
    model="claude-opus-4", 
    messages=[{"role": "user", "content": "Question 2"}],
    task_label="customer-support",  # Same task label
    judge_prompt="""
You are comparing two customer support responses to the same customer inquiry. Is response A better than response B?

Consider these criteria:

HELPFULNESS:
- Does response A address the customer's question more directly than B?
- Does response A provide more actionable next steps or solutions than B?
- Does response A anticipate follow-up questions better than B?

PROFESSIONALISM:
- Does response A use more appropriate tone and language than B?
- Does response A show more empathy and understanding than B?
- Does response A maintain company brand voice better than B?

ACCURACY:
- Is response A's information more factually correct than B's?
- Are response A's policy/procedure references more accurate than B's?

Answer: A is better than B (YES/NO)
"""
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
        judge_prompt="""
You are comparing two customer support responses to the same customer inquiry. Is response A better than response B?

Consider these criteria:

HELPFULNESS:
- Does response A address the customer's question more directly than B?
- Does response A provide more actionable next steps or solutions than B?
- Does response A anticipate follow-up questions better than B?

PROFESSIONALISM:
- Does response A use more appropriate tone and language than B?
- Does response A show more empathy and understanding than B?
- Does response A maintain company brand voice better than B?

ACCURACY:
- Is response A's information more factually correct than B's?
- Are response A's policy/procedure references more accurate than B's?

Answer: A is better than B (YES/NO)
"""
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