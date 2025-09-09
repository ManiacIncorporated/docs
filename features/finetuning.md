# Fine-tuning

Maniac's automated fine-tuning system creates task-specific model adapters (LoRAs) that dramatically improve performance while keeping costs low.

## What is Fine-tuning in Maniac?

Maniac uses Parameter-Efficient Fine-Tuning (PEFT) techniques, specifically LoRA (Low-Rank Adaptation), to create specialized versions of base models for your specific tasks without the cost and complexity of full model training.

### Benefits of Maniac's Fine-tuning:
- **Cost-Effective**: Train adapters instead of full models
- **Fast Training**: Complete training in minutes, not hours
- **Task-Specific**: Optimized for your exact use case
- **Multi-Model**: Train adapters for multiple base models simultaneously
- **Automatic**: Minimal configuration required

## Getting Started

### 1. Prepare Your Training Data

Format your data as JSON with input/output pairs:

```json
[
  {
    "input": "What is 2 + 2?",
    "output": "2 + 2 equals 4."
  },
  {
    "input": "Calculate 15 * 3",
    "output": "15 multiplied by 3 is 45."
  },
  {
    "input": "What's the square root of 16?",
    "output": "The square root of 16 is 4."
  }
]
```

### 2. Start Fine-tuning

```python
from maniac import Maniac

client = Maniac(api_key="your-maniac-api-key")

# Start fine-tuning job
job = client.create_finetuning_job(
    name="math-solver-v1",
    container_id="your-container",
    training_data="training_data.json",
    task_type="question_answering",
    base_models=["llama-2-7b", "mistral-7b", "code-llama-7b"]
)

print(f"Fine-tuning job created: {job.id}")
```

### 3. Monitor Progress

```python
# Check job status
status = client.get_finetuning_status(job.id)
print(f"Status: {status.state}")  # training, completed, failed
print(f"Progress: {status.progress}%")

# Get detailed metrics
if status.state == "completed":
    metrics = client.get_finetuning_metrics(job.id)
    print(f"Final loss: {metrics.final_loss}")
    print(f"Validation accuracy: {metrics.validation_accuracy}")
```

## Advanced Fine-tuning Configuration

### Custom Training Parameters

Fine-tune the training process:

```python
from maniac import FinetuningConfig

config = FinetuningConfig(
    # Training hyperparameters
    learning_rate=5e-5,
    batch_size=16,
    num_epochs=3,
    warmup_steps=100,
    
    # LoRA parameters
    lora_rank=16,
    lora_alpha=32,
    lora_dropout=0.1,
    
    # Optimization settings
    optimizer="adamw",
    lr_scheduler="cosine",
    weight_decay=0.01,
    
    # Early stopping
    early_stopping=True,
    patience=3,
    min_delta=0.001
)

job = client.create_finetuning_job(
    name="custom-training",
    container_id="your-container",
    training_data="data.json",
    config=config
)
```

### Multi-Task Fine-tuning

Train a single adapter for multiple related tasks:

```python
# Multi-task training data
training_data = {
    "math_basic": "math_basic_data.json",
    "math_advanced": "math_advanced_data.json",
    "word_problems": "word_problems_data.json"
}

job = client.create_multitask_finetuning_job(
    name="math-multitask",
    container_id="your-container",
    training_data=training_data,
    task_weights={"math_basic": 0.4, "math_advanced": 0.4, "word_problems": 0.2}
)
```

## Data Management

### Data Quality Analysis

Analyze your training data quality:

```python
# Upload and analyze data
analysis = client.analyze_training_data(
    data_file="training_data.json",
    task_type="question_answering"
)

print(f"Data quality score: {analysis.quality_score}")
print(f"Recommended dataset size: {analysis.recommended_size}")
print(f"Issues found: {len(analysis.issues)}")

for issue in analysis.issues:
    print(f"- {issue.severity}: {issue.description}")
```

### Data Augmentation

Automatically expand your training dataset:

```python
# Enable data augmentation
job = client.create_finetuning_job(
    name="augmented-training",
    container_id="your-container",
    training_data="data.json",
    data_augmentation={
        "enabled": True,
        "methods": ["paraphrasing", "back_translation", "synonym_replacement"],
        "augmentation_ratio": 2.0  # Double the dataset size
    }
)
```

### Incremental Training

Add new data to existing fine-tuned models:

```python
# Continue training with new data
incremental_job = client.create_incremental_training_job(
    base_job_id="previous-job-id",
    new_training_data="additional_data.json",
    learning_rate=1e-5,  # Lower learning rate for incremental updates
    num_epochs=1
)
```

## Model Evaluation

### Automatic Evaluation

Maniac automatically evaluates your fine-tuned models:

```python
# Get evaluation results
evaluation = client.get_model_evaluation(job.id)

print(f"Overall performance: {evaluation.overall_score}")

# Task-specific metrics
for task, metrics in evaluation.task_metrics.items():
    print(f"{task}:")
    print(f"  Accuracy: {metrics.accuracy}")
    print(f"  F1 Score: {metrics.f1_score}")
    print(f"  BLEU Score: {metrics.bleu_score}")
```

### Custom Evaluation Metrics

Define your own evaluation criteria:

```python
from maniac import EvaluationMetric, CustomEvaluator

# Custom evaluator for code generation
code_evaluator = CustomEvaluator(
    name="code_quality",
    metrics=[
        EvaluationMetric.SYNTAX_CORRECTNESS,
        EvaluationMetric.PERFORMANCE_EFFICIENCY,
        EvaluationMetric.CODE_STYLE_COMPLIANCE
    ]
)

# Add to fine-tuning job
job = client.create_finetuning_job(
    name="code-generator",
    container_id="coding-assistant",
    training_data="code_data.json",
    custom_evaluators=[code_evaluator]
)
```

### Human Evaluation

Set up human evaluation workflows:

```python
# Create human evaluation task
human_eval = client.create_human_evaluation(
    job_id=job.id,
    evaluator_pool="expert",  # or "crowd", "internal"
    evaluation_criteria=[
        "Accuracy",
        "Clarity", 
        "Usefulness",
        "Safety"
    ],
    sample_size=100  # Number of responses to evaluate
)

# Check human evaluation results
results = client.get_human_evaluation_results(human_eval.id)
```

## Model Deployment

### Automatic Deployment

Deploy fine-tuned models automatically:

```python
# Enable auto-deployment for successful jobs
job = client.create_finetuning_job(
    name="production-model",
    container_id="your-container",
    training_data="data.json",
    auto_deploy=True,
    deployment_config={
        "environment": "production",
        "traffic_percentage": 10,  # Gradual rollout
        "success_threshold": 0.9
    }
)
```

### Manual Deployment

Control deployment manually:

```python
# Deploy specific model version
deployment = client.deploy_finetuned_model(
    job_id=job.id,
    model_version="v1.2",
    environment="production",
    scaling_config={
        "min_instances": 2,
        "max_instances": 10,
        "target_utilization": 0.8
    }
)

print(f"Deployment ID: {deployment.id}")
print(f"Endpoint URL: {deployment.endpoint_url}")
```

### A/B Testing Deployments

Test fine-tuned models against baselines:

```python
# A/B test fine-tuned vs base model
ab_test = client.create_model_ab_test(
    name="finetuned-vs-base",
    container_id="your-container",
    variants={
        "base": {"model": "llama-2-7b"},
        "finetuned": {"model": f"finetuned-{job.id}"}
    },
    traffic_split={"base": 0.3, "finetuned": 0.7},
    success_metrics=["accuracy", "user_satisfaction", "response_time"]
)
```

## Monitoring Fine-tuned Models

### Performance Monitoring

Track fine-tuned model performance in production:

```python
# Get performance metrics
metrics = client.get_production_metrics(
    container_id="your-container",
    model_version="finetuned-v1",
    time_range="24h"
)

print(f"Request volume: {metrics.request_count}")
print(f"Average latency: {metrics.avg_latency_ms}ms")
print(f"Error rate: {metrics.error_rate}%")
print(f"Quality score: {metrics.avg_quality_score}")
```

### Drift Detection

Monitor for model drift and performance degradation:

```python
# Enable drift detection
client.enable_drift_detection(
    container_id="your-container",
    model_version="finetuned-v1",
    monitoring_config={
        "check_frequency": "hourly",
        "drift_threshold": 0.1,
        "alert_channels": ["email", "slack"]
    }
)

# Check drift status
drift_status = client.get_drift_status("your-container")
if drift_status.drift_detected:
    print(f"Drift detected! Severity: {drift_status.severity}")
    print(f"Recommended action: {drift_status.recommendation}")
```

## Cost Optimization

### Training Cost Analysis

Understand and optimize training costs:

```python
# Get cost breakdown
cost_analysis = client.get_training_costs(job.id)

print(f"Total training cost: ${cost_analysis.total_cost:.2f}")
print(f"Compute cost: ${cost_analysis.compute_cost:.2f}")
print(f"Data storage cost: ${cost_analysis.storage_cost:.2f}")
print(f"Cost per epoch: ${cost_analysis.cost_per_epoch:.2f}")

# Cost optimization recommendations
for rec in cost_analysis.recommendations:
    print(f"💡 {rec.description} (potential savings: ${rec.savings:.2f})")
```

### Resource Optimization

Optimize training resource usage:

```python
from maniac import ResourceConfig

# Efficient resource configuration
resource_config = ResourceConfig(
    instance_type="gpu.t4.medium",  # Cost-effective GPU
    auto_scaling=True,
    spot_instances=True,  # Up to 70% cost savings
    mixed_precision=True,  # Faster training, lower memory
    gradient_checkpointing=True  # Memory optimization
)

job = client.create_finetuning_job(
    name="cost-optimized-training",
    container_id="your-container",
    training_data="data.json",
    resource_config=resource_config
)
```

## Best Practices

### 1. Data Quality First
Invest in high-quality, diverse training data rather than just quantity.

### 2. Start Small
Begin with a smaller dataset and base model, then scale up.

### 3. Monitor Training Progress
Use early stopping and validation metrics to prevent overfitting.

### 4. Gradual Deployment
Deploy with gradual traffic increase to catch issues early.

### 5. Continuous Evaluation
Regularly evaluate model performance and retrain when needed.

### 6. Version Control
Keep track of data versions, model versions, and training configurations.

## Common Fine-tuning Patterns

### Customer Support Agent
```python
job = client.create_finetuning_job(
    name="support-agent",
    container_id="customer-support",
    training_data="support_conversations.json",
    task_type="conversational",
    base_models=["llama-2-7b-chat"],
    config=FinetuningConfig(
        learning_rate=1e-4,
        num_epochs=2,
        lora_rank=8
    )
)
```

### Code Generation Assistant
```python
job = client.create_finetuning_job(
    name="code-assistant",
    container_id="coding-helper",
    training_data="code_examples.json",
    task_type="code_generation",
    base_models=["code-llama-7b"],
    config=FinetuningConfig(
        learning_rate=5e-5,
        num_epochs=3,
        lora_rank=16
    )
)
```

### Domain-Specific QA
```python
job = client.create_finetuning_job(
    name="medical-qa",
    container_id="medical-assistant",
    training_data="medical_qa.json",
    task_type="domain_qa",
    base_models=["llama-2-7b", "mistral-7b"],
    config=FinetuningConfig(
        learning_rate=3e-5,
        num_epochs=4,
        lora_rank=32  # Higher rank for complex domain
    )
)
```

## Troubleshooting

### Common Issues

**Training Loss Not Decreasing**
- Check learning rate (try lower values like 1e-5)
- Verify data quality and format
- Increase batch size or number of epochs

**Overfitting**
- Add dropout to LoRA configuration
- Use early stopping
- Reduce number of training epochs
- Increase validation data size

**Out of Memory Errors**
- Reduce batch size
- Enable gradient checkpointing
- Use smaller LoRA rank
- Try mixed precision training

**Poor Performance**
- Increase training data size
- Check data distribution and balance
- Try different base models
- Adjust LoRA parameters (rank, alpha)

## Next Steps

- [Deploy your fine-tuned models](../guides/deployment.md)
- [Monitor performance in production](../guides/performance.md)
- [Explore the API reference](../api/python-sdk.md)