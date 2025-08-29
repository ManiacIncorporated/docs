# Maniac Optimize: The Performance Engine

**Maniac Optimize** is the component responsible for ensuring that your AI Containers perform at their best, regardless of the underlying model they are deployed on. While a LoRA can make a model an expert on a task, the quality of the prompt is what unlocks its full potential. Maniac Optimize automates the complex process of prompt engineering.

## Purpose

The primary goal of Maniac Optimize is to tune the prompts within an AI Container for maximum effectiveness. It treats prompt engineering as a scientific optimization problem, systematically searching for the best-performing prompt variations. This service is responsible for:

1.  **Prompt Generation**: Creating a diverse set of candidate prompts for a given task.
2.  **Performance Testing**: A/B testing different prompts against a target model to see which ones yield the best results.
3.  **Optimization**: Using the results from testing to refine and improve the prompts iteratively.
4.  **Packaging**: Updating the AI Container with the set of optimized prompts, with specific prompts tailored for different models.

## Architecture

Maniac Optimize is built to industrialize the art of prompt engineering. It uses a combination of model-based generation and rigorous testing to achieve this. The key components include:

-   `main.py`: The central script that likely orchestrates the entire prompt optimization workflow. It takes an AI Container or a task definition and kicks off the process of generating, testing, and selecting the best prompts.

-   `scripts/optimize_modal.py`: This file suggests that Maniac Optimize leverages the [Modal](https://modal.com/) platform to run its optimization jobs. Modal is ideal for this use case, as it allows for massively parallel, on-demand compute. This enables Maniac to test thousands of prompt variations quickly and cost-effectively.

-   `src/models/model_server.py` and `src/models/model_lm.py`: These modules likely define the interfaces for interacting with various language models. The optimization process needs to be model-agnostic, and these files provide a standardized way to send prompts to different models and get responses back.

-   `src/training/trainer.py`: While the name suggests training, in the context of prompt optimization, this module is likely responsible for managing the "training" or "tuning" of the prompts themselves. It orchestrates the iterative process of testing and refining prompt candidates.

-   `src/config/settings.py`: A centralized configuration file for managing settings related to the optimization process, such as model endpoints, API keys, and optimization parameters.

By systematically optimizing the prompts for each task and model, Maniac Optimize ensures that your AI applications are not just portable, but also consistently high-performing. It replaces guesswork and manual tuning with a data-driven, automated process.
