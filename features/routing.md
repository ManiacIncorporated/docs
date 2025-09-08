# Model Selection

Maniac provides intelligent model selection with automatic provider routing. Simply specify your desired model - Maniac handles all provider complexity and failover automatically.

## Model Selection

Choose from a wide range of models across different providers. Maniac automatically routes to the optimal provider for each model:

```python
from maniac import Maniac

# Simple initialization - Maniac handles all provider routing
client = Maniac(api_key="your-maniac-api-key")

# Specify desired model - Maniac handles provider routing automatically
response = client.chat.completions.create(
    fallback="claude-opus-4",
    messages=[{"role": "user", "content": "Hello"}],
    task_label="conversation",
    judge_prompt="""
You are comparing two conversational responses to the same user query. Is response A better than response B?

Consider these criteria:

HELPFULNESS:
- Does response A address the user's needs more appropriately than B?
- Does response A provide more useful information than B?

APPROPRIATENESS:
- Does response A use more suitable tone and language than B?
- Does response A maintain better professional boundaries than B?

Answer: A is better than B (YES/NO)
"""
)
```

## Available Models

### Claude Models (Anthropic via Vertex AI)
- `claude-opus-4` - Most capable model for complex reasoning
- `claude-sonnet-4` - Balanced performance and speed
- `claude-haiku-3` - Fast model for simple tasks

### GPT Models (OpenAI)
- `gpt-4o` - Latest multimodal model
- `gpt-4-turbo` - High performance model  
- `gpt-4` - Classic high-quality model
- `gpt-3.5-turbo` - Fast and cost-effective
- `o1-mini` - Reasoning-optimized model

### Gemini Models (Google)
- `gemini-pro` - Google's advanced model
- `gemini-1.5-pro` - Latest version with long context

### Open Source Models  
- `llama-3.1-70b` - Meta's large language model
- `mixtral-8x7b` - Mistral's mixture-of-experts model
- `codestral` - Specialized for code generation

## Model Selection Strategies

### Performance-First
Use the most capable models for complex tasks:

```python
response = client.responses.create(
    fallback="claude-opus-4",      # Best reasoning capabilities
    input="Complex legal analysis requiring deep understanding...",
    instructions="You are a senior legal counsel analyzing contract risks.",
    task_label="legal-analysis",
    judge_prompt="""
You are comparing two legal analyses. Is response A better than response B?

Consider these criteria:
- Is response A's legal reasoning more sound than B's?
- Does response A identify more potential risks than B?
- Are response A's recommendations more actionable than B's?

Answer: A is better than B (YES/NO)
"""
)
```

### Cost-Optimized
Start with efficient models for simpler tasks:

```python
response = client.responses.create(
    fallback="claude-haiku-3",     # Fast and cost-effective
    input="Simple document summary...",
    instructions="Provide a concise summary of the key points.",
    task_label="document-processing",
    judge_prompt="""
You are comparing two document summaries. Is response A better than response B?

Consider these criteria:
- Does response A capture the key points better than B?
- Is response A more concise while maintaining completeness than B?

Answer: A is better than B (YES/NO)
"""
)
```

### Speed-Optimized
Prioritize low latency for real-time applications:

```python
response = client.responses.create(
    fallback="gpt-3.5-turbo",      # Fast response times
    input="Quick customer inquiry...",
    instructions="Provide a helpful, immediate response to this customer question.",
    task_label="customer-support",
    judge_prompt="""
You are comparing two customer support responses. Is response A better than response B?

Consider these criteria:
- Does response A address the question more directly than B?
- Is response A more helpful while being concise than B?

Answer: A is better than B (YES/NO)
"""
)
```

## Model Information

Get information about available models:

```python
# List all available models
models = client.get_available_models()
print(models)

# Get model details
info = client.get_model_info("claude-opus-4")
print(f"Provider: {info['provider']}")
print(f"Context length: {info['max_context']}")
print(f"Capabilities: {info['capabilities']}")

# Check model availability in real-time
is_available = client.check_model_availability("claude-opus-4")
if is_available:
    print("Model ready for use")
else:
    print("Model temporarily unavailable - automatic failover active")
```

## Best Practices

### 1. Choose Models by Use Case
- **Complex reasoning**: `claude-opus-4`, `gpt-4o`
- **Balanced tasks**: `claude-sonnet-4`, `gpt-4-turbo`  
- **Simple/fast tasks**: `claude-haiku-3`, `gpt-3.5-turbo`
- **Code generation**: `codestral`, `gpt-4o`

### 2. Use Task Labels Consistently
Keep task labels consistent across different model choices:

```python
# Both requests use same task_label for optimization
response1 = client.chat.completions.create(
    fallback="claude-opus-4",
    messages=messages, 
    task_label="legal-review"
)

response2 = client.chat.completions.create(
    fallback="gpt-4o", 
    messages=messages, 
    task_label="legal-review"  # Same task label
)
```

### 3. Monitor Model Usage
Track which models and providers are being used:

```python
response = client.chat.completions.create(
    fallback="claude-opus-4",
    messages=messages,
    task_label="analysis"
)

# Check which model was actually used
print(f"Model used: {response['model']}")
print(f"Provider: {response.get('provider_used', 'unknown')}")
```

### 4. Handle Model Availability
Maniac automatically handles model unavailability, but you can check status:

```python
# Check if your preferred model is available
if client.check_model_availability("claude-opus-4"):
    print("Using claude-opus-4")
else:
    print("claude-opus-4 unavailable, automatic failover in progress")

response = client.chat.completions.create(
    fallback="claude-opus-4",  # Maniac handles failover automatically
    messages=messages,
    task_label="important-task"
)
```

## Environment Configuration

Set your API key via environment variable:

```bash
export MANIAC_API_KEY=your-maniac-api-key
```

Then initialize without parameters:

```python
import os
from maniac import Maniac

client = Maniac(api_key=os.getenv("MANIAC_API_KEY"))
```

## Model Migration

Easily switch between models for experimentation:

```python
# Test different models for the same task
models_to_test = ["claude-opus-4", "gpt-4o", "claude-sonnet-4"]

for model in models_to_test:
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        task_label="analysis-comparison"  # Same task for fair comparison
    )
    print(f"Model {model} response: {response['choices'][0]['message']['content'][:100]}...")
```

## Error Handling

Handle model issues gracefully:

```python
try:
    response = client.chat.completions.create(
        fallback="claude-opus-4",
        messages=messages,
        task_label="critical-task"
    )
except Exception as e:
    print(f"Error with model request: {e}")
    # Maniac handles failover automatically, but log for monitoring
```

## Advanced Features

### Model Performance Insights

```python
# Get performance metrics for different models
metrics = client.get_model_metrics(task_label="your-task")

for model, stats in metrics.items():
    print(f"{model}: avg_latency={stats['latency']}, quality_score={stats['quality']}")
```

### Task-Specific Model Selection

```python
# Different models for different aspects of the same workflow
legal_analysis = client.responses.create(
    fallback="claude-opus-4",    # Best for complex reasoning
    input=contract_text,
    task_label="contract-review"
)

summary = client.responses.create(
    fallback="claude-haiku-3",   # Fast for simple summarization
    input=legal_analysis["output_text"], 
    task_label="contract-summary"
)
```

## Next Steps

- [Learn about the Python SDK](../api/python-sdk.md)
- [Explore task organization](../guides/best-practices.md)
- [Set up batch processing](../guides/batch-processing.md)