# Maniac Finetune: The Model Container Factory

**Maniac Finetune** is the heart of the Maniac platform. It is responsible for creating the **Model Containers** that power your applications. This service takes your raw production data and a base model, and produces a portable, task-specific package that can be deployed across any model in the Maniac ecosystem.

## Purpose

The primary goal of Maniac Finetune is to handle the entire lifecycle of fine-tuning and packaging. It automates the process of:

1.  **Data Processing**: Ingesting and preparing your training data for fine-tuning.
2.  **LoRA Training**: Efficiently fine-tuning a base model to create a specialized LoRA (Low-Rank Adaptation) for your specific task.
3.  **Evaluation**: Judging the performance of the newly trained artifact to ensure it meets quality standards.
4.  **Packaging**: Bundling the LoRA, system prompts, and other metadata into a standardized Model Container format.

## Architecture

Maniac Finetune is designed as a modular pipeline that orchestrates the creation of Model Containers. The key components include:

-   `finetune_app.py`: The main entry point for the finetuning service. It likely exposes an API to trigger new training jobs and manage the lifecycle of Model Containers.

-   `data_handler.py`: A crucial utility for ingesting, cleaning, and formatting the training data. It prepares the dataset for the model training process, ensuring the model learns from high-quality examples.

-   `ft_model.py`: This module defines the core model architecture used for fine-tuning. It incorporates techniques like LoRA to ensure that the training process is efficient and the resulting artifacts are lightweight.

-   `train_and_evaluate.py`: This script orchestrates the end-to-end process. It takes the prepared data and the model definition, runs the training loop, and then kicks off an evaluation process to score the new model's performance.

-   `judge.py`: After a model is trained, this component evaluates its outputs. It uses a sophisticated "judge" prompt to score the quality of the model's responses, providing a quantitative measure of its capabilities. This is critical for the Router's ranking system.

-   `supabase_utils.py`: This utility suggests that Maniac Finetune integrates with a Supabase backend to store metadata about the training jobs, model artifacts, and evaluation results. This provides a persistent record of all Model Containers created by the system.

By combining these components, Maniac Finetune provides a robust, automated factory for producing expert, task-specific AI programs that are ready for deployment by the Maniac Router.
