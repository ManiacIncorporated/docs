# Maniac: The Kubernetes for AI

Welcome to Maniac, an open-source platform for deploying and managing performant AI-powered applications with flexibility. Maniac brings the principles of containerization and orchestration to AI, allowing you to treat AI models like cloud servers: interchangeable, scalable, and adaptable.

## The Core Analogy: Kubernetes for AI

In modern software development, Kubernetes allows you to package an application and its dependencies into a single, portable container that can be deployed on any server, in any cloud. This eliminates vendor lock-in and provides massive scalability.

Maniac applies this same revolutionary concept to AI. Instead of building your application for a single, monolithic AI model (like GPT-4 or Claude 3), Maniac lets you package your AI-driven task into a portable **Model Container**.

## What is a Model Container?

A Model Container is a self-contained package that holds everything required to execute a specific AI task. It's an abstract AI program, much like a DSPy program, that can be deployed across any supported Large Language Model. 

Each container bundles:

- **Task-Specific LoRAs**: Lightweight, fine-tuned adapters that make any base model an expert on your specific task.
- **Optimized Prompts**: A set of prompts that are individually optimized for different models, ensuring the best performance no matter where the container is deployed.
- **System Prompt**: The core instructions that define the task and the AI's persona or objective.
- **Judge Prompt**: A specialized prompt used to evaluate the model's output, enabling continuous, automated quality control.

## How It Works

The Maniac ecosystem is composed of three key services:

1.  **Maniac Finetune**: The "container factory." This service trains the LoRAs and prepares all the components that make up an Model Container.
2.  **Maniac Optimize**: The performance engine. This service optimizes the prompts within a container to ensure they are maximally effective for each target model.
3.  **Maniac Router**: The "control plane" or orchestrator. The Router intelligently deploys Model Containers to the best-suited model from a cluster of available options. It constantly evaluates model performance and routes tasks dynamically, ensuring your application always runs on the most effective and efficient infrastructure.

By decoupling the AI task from the underlying model, Maniac empowers you to build applications that are:

- **Model-Agnostic**: Swap models in and out without changing your application code.
- **Future-Proof**: Seamlessly integrate new, state-of-the-art models as they become available.
- **Cost-Effective**: Route tasks to the most cost-efficient model that meets your performance requirements.
- **Highly Performant**: Automatically use the best-performing model for every task, every time.

This documentation will guide you through the architecture of Maniac and show you how to start building your own Model Containers.
