# Quick Start Guide

Get up and running with Maniac in under 5 minutes.

## Prerequisites

- Python 3.8 or higher
- An API key from the Maniac platform

## Installation

Install the Maniac Python SDK using pip:

```bash
pip install maniac-ai
```

## Basic Setup

### 1. Import and Initialize

```python
from maniac import ManiacClient

# Initialize the client with your API key
client = ManiacClient(
    api_key="your-api-key-here",
    environment="production"  # or "sandbox" for testing
)
```

### 2. Create Your First Model Container

```python
# Define your task
container = client.create_container(
    name="math-solver",
    task_type="math_problem_solving",
    description="Solves mathematical equations and word problems"
)

# The platform will automatically:
# - Select optimal base models
# - Generate task-specific prompts
# - Begin continuous optimization
```

### 3. Make Your First Request

```python
# Send a request to your optimized container
response = client.complete(
    container_id="math-solver",
    prompt="A train travels 120 miles in 2 hours. What is its average speed?",
    optimize=True  # Enable automatic optimization
)

print(response.text)
# Output: "The average speed is 60 miles per hour."

# View performance metrics
print(f"Model used: {response.model}")
print(f"Latency: {response.latency_ms}ms")
print(f"Cost: ${response.cost:.4f}")
```

## Next Steps

Now that you've made your first optimized request, explore more features:

- [Configure multiple models](../features/routing.md) for intelligent routing
- [Fine-tune models](../features/finetuning.md) on your specific data
- [Monitor performance](../guides/performance.md) and optimize costs

## Example: Building a Customer Support Agent

Here's a complete example of building an optimized customer support agent:

```python
from maniac import ManiacClient

client = ManiacClient(api_key="your-api-key")

# Create a specialized container for customer support
support_container = client.create_container(
    name="customer-support",
    task_type="conversation",
    config={
        "tone": "friendly and professional",
        "knowledge_base": "support_docs.json",
        "languages": ["en", "es", "fr"]
    }
)

# Process customer queries
def handle_customer_query(query: str, customer_id: str):
    response = client.complete(
        container_id="customer-support",
        prompt=query,
        context={
            "customer_id": customer_id,
            "history": client.get_conversation_history(customer_id)
        },
        optimize=True
    )
    
    # Log for continuous improvement
    client.log_interaction(
        container_id="customer-support",
        prompt=query,
        response=response.text,
        customer_id=customer_id
    )
    
    return response.text

# Use in your application
answer = handle_customer_query(
    "How do I reset my password?",
    customer_id="cust_123"
)
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