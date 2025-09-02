# Model Optimization

Maniac's optimization engine automatically improves model performance for your specific tasks through prompt optimization, parameter tuning, and intelligent caching strategies.

## What is Model Optimization?

Model optimization in Maniac involves several automated techniques:

- **Prompt Engineering**: Automatically optimize prompts for better results
- **Few-Shot Learning**: Dynamically select the best examples for in-context learning
- **Parameter Tuning**: Optimize sampling parameters (temperature, top-p, etc.)
- **Response Formatting**: Ensure consistent, structured outputs
- **Context Management**: Optimize context length and content

## Getting Started with Optimization

### Basic Optimization

Enable optimization for any request:

```python
from maniac import ManiacClient

client = ManiacClient(api_key="your-key")

# Basic optimization enabled
response = client.complete(
    container_id="your-container",
    prompt="Solve this math problem: 2x + 5 = 13",
    optimize=True  # Enable automatic optimization
)

print(response.text)
print(f"Optimization score: {response.optimization_score}")
```

### Optimization Strategies

Choose from different optimization approaches:

```python
from maniac import OptimizationStrategy

# Performance optimization
response = client.complete(
    container_id="your-container",
    prompt="Write a creative story",
    optimization_strategy=OptimizationStrategy.QUALITY_FOCUSED
)

# Speed optimization
response = client.complete(
    container_id="your-container", 
    prompt="Quick factual answer",
    optimization_strategy=OptimizationStrategy.LATENCY_FOCUSED
)

# Balanced optimization
response = client.complete(
    container_id="your-container",
    prompt="Analyze this data",
    optimization_strategy=OptimizationStrategy.BALANCED
)
```

## Prompt Optimization

### Automatic Prompt Enhancement

Maniac automatically improves your prompts:

```python
# Your original prompt
original_prompt = "Write code"

# Maniac optimizes it automatically
response = client.complete(
    container_id="coding-assistant",
    prompt=original_prompt,
    optimize_prompt=True,
    show_optimized_prompt=True
)

print("Original:", original_prompt)
print("Optimized:", response.optimized_prompt)
# Optimized: "Write clean, well-commented code following best practices. 
# Include error handling and type hints where appropriate."
```

### Template-Based Optimization

Create reusable optimized templates:

```python
# Create an optimized template
template = client.create_template(
    name="code_review",
    base_prompt="Review this code: {code}",
    optimization_goals=["clarity", "constructiveness", "actionability"],
    task_type="code_review"
)

# Use the optimized template
response = client.complete_with_template(
    template_id=template.id,
    variables={"code": "def hello(): print('world')"},
    container_id="your-container"
)
```

## Few-Shot Learning Optimization

### Dynamic Example Selection

Automatically select the best examples for your task:

```python
# Provide a pool of examples
examples = [
    {"input": "What's 2+2?", "output": "2+2 equals 4"},
    {"input": "Calculate 15*3", "output": "15 multiplied by 3 is 45"},
    {"input": "What's the square root of 16?", "output": "The square root of 16 is 4"},
    # ... more examples
]

response = client.complete(
    container_id="math-solver",
    prompt="What's 7*8?",
    examples_pool=examples,
    auto_select_examples=True,  # Automatically pick best examples
    max_examples=3
)
```

### Example Quality Scoring

Maniac scores and ranks your examples:

```python
# Analyze example quality
example_scores = client.analyze_examples(
    container_id="your-container",
    examples=examples,
    task_type="math_problem"
)

for example, score in example_scores.items():
    print(f"Example quality: {score:.2f} - {example['input'][:50]}")
```

## Parameter Optimization

### Automatic Parameter Tuning

Optimize sampling parameters for your specific use case:

```python
# Enable automatic parameter optimization
response = client.complete(
    container_id="your-container",
    prompt="Generate creative content",
    optimize_parameters=True,
    parameter_bounds={
        "temperature": (0.1, 1.0),
        "top_p": (0.8, 1.0),
        "max_tokens": (100, 500)
    }
)

print(f"Optimized parameters: {response.optimized_parameters}")
# Output: {"temperature": 0.7, "top_p": 0.9, "max_tokens": 250}
```

### Custom Parameter Sets

Define parameter sets for different scenarios:

```python
# Create parameter profiles
creative_params = {
    "temperature": 0.8,
    "top_p": 0.9,
    "frequency_penalty": 0.1
}

analytical_params = {
    "temperature": 0.2,
    "top_p": 0.8,
    "frequency_penalty": 0.0
}

# Use appropriate parameters based on task
if task_type == "creative_writing":
    params = creative_params
else:
    params = analytical_params

response = client.complete(
    container_id="your-container",
    prompt=prompt,
    parameters=params
)
```

## Response Formatting Optimization

### Structured Output Optimization

Ensure consistent, well-formatted responses:

```python
from maniac import OutputFormat, ValidationRule

# Define desired output format
output_format = OutputFormat(
    type="json",
    schema={
        "answer": "string",
        "confidence": "float",
        "reasoning": "string"
    },
    validation_rules=[
        ValidationRule.REQUIRED_FIELDS(["answer", "confidence"]),
        ValidationRule.RANGE("confidence", 0.0, 1.0)
    ]
)

response = client.complete(
    container_id="qa-system",
    prompt="What is the capital of France?",
    output_format=output_format,
    optimize_formatting=True
)

# Response will be properly formatted JSON
result = response.json()
print(f"Answer: {result['answer']}")
print(f"Confidence: {result['confidence']}")
```

### Format Consistency

Maintain consistent formatting across requests:

```python
# Enable format learning
client.enable_format_learning(
    container_id="your-container",
    learn_from_history=True,  # Learn from past successful formats
    consistency_threshold=0.9
)

# Subsequent requests will maintain learned format patterns
response = client.complete(
    container_id="your-container",
    prompt="Another question",
    maintain_format_consistency=True
)
```

## Context Optimization

### Intelligent Context Management

Optimize context window usage:

```python
# Automatic context optimization
response = client.complete(
    container_id="your-container",
    prompt="Analyze this long document",
    context=long_document_text,
    optimize_context=True,  # Automatically optimize context usage
    context_compression="intelligent"  # Use smart compression
)

print(f"Original context length: {len(long_document_text)}")
print(f"Optimized context length: {response.optimized_context_length}")
```

### Dynamic Context Selection

Select the most relevant context automatically:

```python
# Multiple context sources
context_sources = [
    {"text": document1, "relevance_weight": 1.0},
    {"text": document2, "relevance_weight": 0.8}, 
    {"text": document3, "relevance_weight": 0.6}
]

response = client.complete(
    container_id="document-qa",
    prompt="What are the main conclusions?",
    context_sources=context_sources,
    auto_select_context=True,  # Automatically select most relevant
    max_context_tokens=2000
)
```

## Optimization Analytics

### Performance Tracking

Monitor optimization effectiveness:

```python
# Get optimization metrics
metrics = client.get_optimization_metrics(
    container_id="your-container",
    time_range="7d"
)

print(f"Average improvement: {metrics['avg_improvement']}%")
print(f"Success rate: {metrics['success_rate']}%")
print(f"Cost impact: {metrics['cost_change']}%")

# Breakdown by optimization type
for opt_type, improvement in metrics['by_optimization_type'].items():
    print(f"{opt_type}: {improvement}% improvement")
```

### A/B Testing Optimization

Test different optimization strategies:

```python
# Create optimization A/B test
test = client.create_optimization_test(
    container_id="your-container",
    name="prompt_optimization_test",
    variants={
        "control": {"optimize": False},
        "treatment": {
            "optimize": True,
            "optimization_strategy": OptimizationStrategy.QUALITY_FOCUSED
        }
    },
    success_metric="user_satisfaction",
    duration_days=14
)

# Results will be automatically analyzed
results = client.get_test_results(test.id)
```

## Advanced Optimization

### Multi-Objective Optimization

Optimize for multiple goals simultaneously:

```python
from maniac import OptimizationGoal, MultiObjectiveConfig

# Define multiple optimization goals
config = MultiObjectiveConfig(
    goals=[
        OptimizationGoal.ACCURACY(weight=0.4),
        OptimizationGoal.LATENCY(weight=0.3),
        OptimizationGoal.COST(weight=0.3)
    ],
    optimization_method="pareto_optimal"
)

response = client.complete(
    container_id="your-container",
    prompt="Complex analysis task",
    multi_objective_config=config
)
```

### Reinforcement Learning from Human Feedback (RLHF)

Continuously improve based on user feedback:

```python
# Enable RLHF optimization
client.enable_rlhf(
    container_id="your-container",
    feedback_collection=True,
    learning_rate=0.1,
    update_frequency="daily"
)

# Provide feedback on responses
response = client.complete(
    container_id="your-container",
    prompt="Generate a summary"
)

# User provides feedback
client.submit_feedback(
    response_id=response.id,
    rating=4,  # 1-5 scale
    feedback_text="Good summary but could be more concise"
)
```

## Best Practices

### 1. Start with Basic Optimization
Enable basic optimization first, then add specific optimizations as needed.

### 2. Monitor Performance Impact
Track how optimization affects quality, latency, and costs.

### 3. Use Task-Specific Optimization
Different task types benefit from different optimization strategies.

### 4. Provide Quality Examples
High-quality examples lead to better few-shot optimization.

### 5. Regular Optimization Review
Periodically review and update your optimization strategies.

## Common Optimization Patterns

### Content Generation
```python
response = client.complete(
    container_id="content-generator",
    prompt="Write a blog post about AI",
    optimization_strategy=OptimizationStrategy.QUALITY_FOCUSED,
    optimize_prompt=True,
    parameters={"temperature": 0.8}
)
```

### Data Analysis
```python
response = client.complete(
    container_id="data-analyst",
    prompt="Analyze this dataset",
    optimization_strategy=OptimizationStrategy.ACCURACY_FOCUSED,
    output_format=structured_format,
    optimize_context=True
)
```

### Customer Support
```python
response = client.complete(
    container_id="support-agent",
    prompt="Help with this issue",
    optimization_strategy=OptimizationStrategy.BALANCED,
    optimize_parameters=True,
    maintain_format_consistency=True
)
```

## Next Steps

- [Learn about fine-tuning](finetuning.md)
- [Set up performance monitoring](../guides/performance.md)
- [Explore the API reference](../api/python-sdk.md)