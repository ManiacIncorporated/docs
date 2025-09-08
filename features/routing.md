# Model Selection & Fallbacks

Maniac provides intelligent model selection with automatic provider routing and fallback capabilities. Simply specify your desired model and optional fallback - Maniac handles all provider complexity automatically.

## Model Selection

Choose from a wide range of models across different providers. Maniac automatically routes to the optimal provider for each model:

```python
from maniac import Maniac

# Simple initialization - Maniac handles all provider routing
client = Maniac(api_key="your-maniac-api-key")

# Use any model with automatic provider routing
response = client.chat.completions.create(
    model="claude-opus-4",    # Maniac routes to Anthropic via Vertex AI
    fallback="gpt-4o",        # Automatic fallback to OpenAI if needed
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

## Fallback Strategy

Configure automatic fallbacks for reliability. Maniac will seamlessly switch to your fallback model if the primary is unavailable:

### Basic Fallback

```python
# Single fallback
response = client.responses.create(
    model="claude-opus-4",
    fallback="gpt-4o",
    input="Analyze this financial data",
    instructions="You are a financial analyst",
    task_label="financial-analysis",
    judge_prompt="""
You are comparing two financial analyses. Is response A better than response B?

Consider these criteria:
- Is response A's analysis more thorough than B's?
- Is response A's reasoning more sound than B's?
- Are response A's recommendations more actionable than B's?

Answer: A is better than B (YES/NO)
"""
)
```

### Multiple Fallback Options

```python
# Multiple fallbacks for maximum reliability
response = client.chat.completions.create(
    model="claude-opus-4",
    fallback=["gpt-4o", "gemini-pro", "llama-3.1-70b"],  # Try in order
    messages=[{"role": "user", "content": "Critical business decision analysis"}],
    task_label="business-analysis",
    judge_prompt="""
You are comparing two business analyses. Is response A better than response B?

Consider these criteria:
- Does response A provide better strategic insights than B?
- Is response A's risk assessment more comprehensive than B's?
- Are response A's recommendations more practical than B's?

Answer: A is better than B (YES/NO)
"""
)
```

## Model Selection Strategies

### Performance-First
Use the most capable models with fast fallbacks:

```python
response = client.responses.create(
    model="claude-opus-4",      # Best reasoning
    fallback="gpt-4o",          # Strong alternative
    input="Complex legal analysis...",
    task_label="legal-analysis"
)
```

### Cost-Optimized
Start with efficient models, fallback to premium when needed:

```python
response = client.responses.create(
    model="claude-haiku-3",     # Fast and cost-effective
    fallback="claude-sonnet-4", # Better quality fallback
    input="Simple document summary...",
    task_label="document-processing"
)
```

### Speed-Optimized
Prioritize low latency:

```python
response = client.responses.create(
    model="gpt-3.5-turbo",      # Fast response
    fallback="claude-haiku-3",  # Also fast
    input="Quick customer inquiry...",
    task_label="customer-support"
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

# Check model availability
is_available = client.check_model_availability("claude-opus-4")
if not is_available:
    print("Primary model unavailable, will use fallback")
```

## Best Practices

### 1. Choose Models by Use Case
- **Complex reasoning**: `claude-opus-4`, `gpt-4o`
- **Balanced tasks**: `claude-sonnet-4`, `gpt-4-turbo`  
- **Simple/fast tasks**: `claude-haiku-3`, `gpt-3.5-turbo`
- **Code generation**: `codestral`, `gpt-4o`

### 2. Configure Appropriate Fallbacks
```python
# Good fallback strategy (similar capabilities)
model="claude-opus-4", fallback="gpt-4o"

# Poor fallback strategy (very different capabilities)
model="claude-opus-4", fallback="gpt-3.5-turbo"
```

### 3. Use Task Labels Consistently
Keep task labels consistent regardless of model choice:

```python
# Both requests use same task_label for optimization
response1 = client.chat.completions.create(
    model="claude-opus-4", fallback="gpt-4o",
    messages=messages, task_label="legal-review"
)

response2 = client.chat.completions.create(
    model="gpt-4o", fallback="claude-sonnet-4", 
    messages=messages, task_label="legal-review"  # Same task label
)
```

### 4. Monitor Model Usage
Track which models are being used:

```python
response = client.chat.completions.create(
    model="claude-opus-4",
    fallback="gpt-4o",
    messages=messages,
    task_label="analysis"
)

# Check which model was actually used
print(f"Model used: {response['model']}")
print(f"Was fallback used: {response.get('fallback_used', False)}")
```

## Environment Variables

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

Easily switch between models:

```python
# Before
response = client.chat.completions.create(
    model="claude-sonnet-4",
    messages=messages,
    task_label="analysis"
)

# After (upgrade to more capable model)
response = client.chat.completions.create(
    model="claude-opus-4",
    fallback="claude-sonnet-4",  # Keep old model as fallback
    messages=messages,
    task_label="analysis"  # Same task label for optimization continuity
)
```

## Error Handling

Handle model availability issues gracefully:

```python
try:
    response = client.chat.completions.create(
        model="claude-opus-4",
        fallback="gpt-4o",
        messages=messages,
        task_label="critical-task"
    )
except Exception as e:
    print(f"Error with primary and fallback models: {e}")
    # Implement additional error handling as needed
```

## Next Steps

- [Learn about the Python SDK](../api/python-sdk.md)
- [Explore task organization](../guides/best-practices.md)
- [Set up batch processing](../guides/batch-processing.md)