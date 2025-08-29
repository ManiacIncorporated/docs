# Maniac Router: The AI Control Plane

**Maniac Router** is the orchestrator or "control plane" of the Maniac platform. It is the component that brings the "Kubernetes for AI" analogy to life. The Router takes an incoming request and an AI Container, and intelligently deploys it to the best possible model from a dynamic cluster of options. 

## Purpose

The primary purpose of the Maniac Router is to decouple the application from the model infrastructure. It handles the complex task of model selection, allowing your application to simply request a task to be completed by the best available AI. Its core responsibilities are:

1.  **Model Ranking**: Continuously evaluating and ranking the performance of all available models (both base models and fine-tuned LoRAs) for various tasks.
2.  **Intelligent Routing**: Selecting the optimal model for a given task based on performance, cost, and other business logic.
3.  **Dynamic Deployment**: Deploying the specified AI Container to the chosen model and returning the result to the application.
4.  **Load Balancing**: Distributing requests across the model cluster to ensure high availability and performance.

## Architecture

Maniac Router uses a sophisticated, data-driven system to manage the model cluster. It treats model selection not as a static choice, but as a continuous optimization problem. The key components include:

-   `main.py`: The main entry point for the Router service. It exposes an API that your applications can call to submit tasks. This API endpoint is the single point of contact for executing any AI task on the Maniac platform.

-   `elo.py`: This is the core of the Router's intelligence. This file implements an **Elo rating system** to rank the performance of different models. Much like in chess, models are ranked against each other based on the quality of their outputs. When a model produces a better output for a given task, its Elo score increases, making it more likely to be chosen for similar tasks in the future.

-   `judge_utils.py`: This utility provides the data that fuels the Elo ranking system. It contains functions to programmatically "judge" or score the outputs from different models. This continuous feedback loop of judging and ranking allows the Router to adapt to changes in model performance over time.

-   `supabase_utils.py`: The Router needs a persistent, real-time backend to store its model rankings and other operational data. This utility manages the connection to a Supabase database, where the Elo ratings and performance metrics are stored. This ensures that the routing intelligence is always up-to-date.

By combining a dynamic Elo-based ranking system with a robust API, the Maniac Router provides a powerful control plane for your AI applications. It abstracts away the complexity of managing a diverse and evolving cluster of models, allowing you to focus on building great products.
