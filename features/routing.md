# Model Routing

Maniac's intelligent routing system automatically selects the optimal model for each request based on performance, cost, and latency requirements.

## How Routing Works

The routing engine continuously evaluates model performance across multiple dimensions:

- **Task Performance**: Accuracy and quality for specific task types
- **Response Time**: Latency from request to response
- **Cost Efficiency**: Price per token and total request cost
- **Availability**: Model uptime and queue length

## Routing Strategies

### 1. Performance-First Routing

Prioritizes the highest-quality responses regardless of cost:

```python
from maniac import ManiacClient, RoutingStrategy

client = ManiacClient(api_key="your-key")

response = client.complete(
    container_id="your-container",
    prompt="Explain quantum computing",
    routing_strategy=RoutingStrategy.PERFORMANCE_FIRST
)

print(f"Used model: {response.model}")  # e.g., "gpt-4-turbo"
print(f"Quality score: {response.quality_score}")  # e.g., 0.95
```

### 2. Cost-Optimized Routing

Balances performance with cost efficiency:

```python
response = client.complete(
    container_id="your-container",
    prompt="Summarize this article",
    routing_strategy=RoutingStrategy.COST_OPTIMIZED,
    max_cost=0.01  # Maximum cost per request
)

print(f"Used model: {response.model}")  # e.g., "claude-3-haiku"
print(f"Cost: ${response.cost:.4f}")  # e.g., $0.0085
```

### 3. Latency-Optimized Routing

Minimizes response time:

```python
response = client.complete(
    container_id="your-container",
    prompt="Quick factual question",
    routing_strategy=RoutingStrategy.LATENCY_FIRST,
    max_latency_ms=500  # Maximum acceptable latency
)

print(f"Response time: {response.latency_ms}ms")  # e.g., 230ms
```

### 4. Custom Routing

Define your own routing criteria:

```python
from maniac import CustomRoutingConfig

custom_config = CustomRoutingConfig(
    performance_weight=0.4,
    cost_weight=0.3,
    latency_weight=0.3,
    minimum_quality_threshold=0.8
)

response = client.complete(
    container_id="your-container",
    prompt="Complex reasoning task",
    routing_config=custom_config
)
```

## Model Pool Configuration

### Adding Models to Your Pool

Configure which models are available for routing:

```python
# Configure available models
client.configure_model_pool(
    container_id="your-container",
    models=[
        {
            "name": "gpt-4-turbo",
            "provider": "openai",
            "max_tokens": 4096,
            "cost_per_token": 0.00003
        },
        {
            "name": "claude-3-opus",
            "provider": "anthropic",
            "max_tokens": 4096,
            "cost_per_token": 0.000015
        },
        {
            "name": "gemini-pro",
            "provider": "google",
            "max_tokens": 2048,
            "cost_per_token": 0.00001
        }
    ]
)
```

### Model Constraints

Set constraints for specific models:

```python
# Only use specific models for sensitive data
response = client.complete(
    container_id="your-container",
    prompt="Process confidential information",
    allowed_models=["claude-3-opus"],  # Only use this model
    require_encryption=True
)

# Exclude certain models
response = client.complete(
    container_id="your-container",
    prompt="General query",
    excluded_models=["gpt-3.5-turbo"]  # Don't use this model
)
```

## Routing Analytics

### Real-time Routing Decisions

Monitor routing decisions in real-time:

```python
# Get routing explanation
response = client.complete(
    container_id="your-container",
    prompt="Your query here",
    explain_routing=True
)

print(response.routing_explanation)
# Output: {
#   "selected_model": "claude-3-sonnet",
#   "reason": "Best balance of cost and performance",
#   "alternatives_considered": ["gpt-4", "gemini-pro"],
#   "decision_factors": {
#     "performance_score": 0.89,
#     "cost_score": 0.95,
#     "latency_score": 0.78
#   }
# }
```

### Historical Routing Data

Analyze routing patterns over time:

```python
# Get routing statistics
stats = client.get_routing_stats(
    container_id="your-container",
    time_range="7d"
)

print(f"Most used model: {stats['top_model']}")
print(f"Average cost: ${stats['avg_cost']:.4f}")
print(f"Average latency: {stats['avg_latency_ms']}ms")

# Model usage distribution
for model, percentage in stats['model_distribution'].items():
    print(f"{model}: {percentage}% of requests")
```

## Advanced Routing Features

### Fallback Chains

Configure automatic fallbacks when primary models fail:

```python
fallback_config = {
    "primary": "gpt-4-turbo",
    "fallbacks": [
        {"model": "claude-3-opus", "condition": "rate_limit"},
        {"model": "gemini-pro", "condition": "any_error"}
    ],
    "max_retries": 3
}

response = client.complete(
    container_id="your-container",
    prompt="Critical request",
    fallback_config=fallback_config
)
```

### A/B Testing

Test different routing strategies:

```python
# Split traffic between strategies
client.create_ab_test(
    container_id="your-container",
    test_name="routing_optimization",
    variants={
        "control": RoutingStrategy.COST_OPTIMIZED,
        "treatment": RoutingStrategy.PERFORMANCE_FIRST
    },
    traffic_split={"control": 0.5, "treatment": 0.5},
    duration_days=14
)
```

### Geographic Routing

Route based on user location for optimal latency:

```python
response = client.complete(
    container_id="your-container",
    prompt="Location-aware request",
    user_location="us-west",  # Routes to nearby models
    prefer_local_models=True
)
```

## Performance Optimization

### Caching and Routing

Combine caching with intelligent routing:

```python
response = client.complete(
    container_id="your-container",
    prompt="Frequently asked question",
    enable_cache=True,
    cache_ttl=3600,  # Cache for 1 hour
    routing_strategy=RoutingStrategy.CACHE_FIRST  # Check cache before routing
)

if response.cache_hit:
    print("Response served from cache")
else:
    print(f"Response generated by {response.model}")
```

### Batch Routing

Optimize routing for batch requests:

```python
prompts = [
    "Question 1",
    "Question 2", 
    "Question 3"
]

# Batch processing with optimized routing
responses = client.complete_batch(
    container_id="your-container",
    prompts=prompts,
    batch_routing=True,  # Optimize routing across the entire batch
    max_parallel=5
)

for i, response in enumerate(responses):
    print(f"Prompt {i+1} used {response.model}")
```

## Monitoring and Alerts

### Setting Up Alerts

Configure alerts for routing issues:

```python
# Set up cost alerts
client.create_alert(
    type="cost_threshold",
    threshold=100.0,  # Alert when daily cost exceeds $100
    container_id="your-container"
)

# Set up performance alerts
client.create_alert(
    type="quality_degradation",
    threshold=0.8,  # Alert when quality drops below 80%
    container_id="your-container"
)
```

### Routing Health Dashboard

Access routing metrics programmatically:

```python
health = client.get_routing_health(container_id="your-container")

print(f"Routing health score: {health['score']}")
print(f"Active models: {health['active_models']}")
print(f"Failed requests: {health['error_rate']}%")

if health['recommendations']:
    print("Recommendations:")
    for rec in health['recommendations']:
        print(f"- {rec}")
```

## Best Practices

### 1. Start Simple
Begin with default routing and gradually customize based on your needs.

### 2. Monitor Performance
Regularly review routing decisions and adjust strategies based on actual usage patterns.

### 3. Set Appropriate Constraints
Use cost and latency limits to prevent unexpected charges or poor user experience.

### 4. Test Different Strategies
Use A/B testing to find the optimal routing strategy for your use case.

### 5. Plan for Failures
Always configure fallback models to ensure service reliability.

## Next Steps

- [Learn about model optimization](optimization.md)
- [Set up fine-tuning](finetuning.md)
- [Monitor performance](../guides/performance.md)