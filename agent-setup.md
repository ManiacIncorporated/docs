# Maniac Agent Setup

You are instrumenting a codebase to log LLM inference data to **Maniac**, an enterprise AI platform for fine-tuning task-specific models. Follow these instructions precisely.

> **Important:** This integration only _records_ LLM completions for training data collection. It does **not** modify, intercept, or affect existing inference in any way. Your application's behavior, outputs, and costs remain completely unchanged.

---

## Step 1 — Scan the codebase

Find all files that make LLM / AI API calls. Look for:

- OpenAI client usage (`openai`, `@ai-sdk/openai`, `OpenAI`)
- Anthropic client usage (`anthropic`, `@ai-sdk/anthropic`, `Anthropic`)
- Google / Gemini usage
- Vercel AI SDK usage (`ai`, `generateText`, `streamText`, `generateObject`)
- Any HTTP calls to `api.openai.com`, `api.anthropic.com`, or similar LLM endpoints
- Other LLM SDKs (Cohere, Mistral, LiteLLM, etc.)

List every call site you find. Then pick the **single best candidate for a Small Language Model (SLM)**. Good indicators are:

- **Text-in, text-out** — the input and output are primarily natural language (not tool-use-heavy or multi-modal)
- **High throughput** — the call happens frequently or at scale
- **Quality-sensitive** — getting the output right matters a lot for the product
- **Repetitive patterns** — similar inputs recur often, which SLMs learn well

If none of these stand out clearly, SLMs are still generally better on cost and latency — just pick the call you think is the best fit. You will instrument only this one call site.

---

## Step 2 — Determine the language

Check whether the call site you picked is **Python** or **TypeScript / JavaScript**, then follow the corresponding section below.

---

## Python integration

### 2a. Install the SDK

Add `maniac` to the project's dependencies:

```bash
pip install maniac
```

If the project uses a `requirements.txt`, add `maniac` to it. If it uses `pyproject.toml` or `setup.py`, add it there instead.

### 2b. Set the environment variable

Add `MANIAC_API_KEY` to the project's `.env` file (or `.env.example` / `.env.local` if that is the convention in this repo). Use a placeholder value:

```
MANIAC_API_KEY=your-maniac-api-key
```

### 2c. Create a container and register completions

Add the following to the call site you identified. This creates a Maniac container (a task namespace that stores inference logs) and registers completions **after** the existing LLM call. Do not remove or modify the existing call.

Set `CONTAINER_LABEL` to a short, descriptive kebab-case label for the task (e.g. `"support-chat"`, `"code-review"`, `"summarizer"`). Set `INITIAL_MODEL` to the model the existing code is using (e.g. `"openai/gpt-4o"`, `"anthropic/claude-sonnet-4"`). If Maniac does not support that model yet, it will throw an error — in that case, just substitute any supported model as a placeholder (e.g. `"openai/gpt-4o"`).

```python
import os
from maniac import Maniac

maniac = Maniac(api_key=os.getenv("MANIAC_API_KEY"))

# Create the container once
maniac.containers.create(
    label="CONTAINER_LABEL",
    initial_model="INITIAL_MODEL",
)

# --- existing code that calls the LLM ---
# response = your_existing_llm_call(...)
# ---

# Register the completion with Maniac (logging only — does not affect inference)
maniac.chat.completions.register(
    container="CONTAINER_LABEL",
    items=[{
        "input": {
            "messages": [{"role": "user", "content": user_input}]
        },
        "output": {
            "choices": [{
                "message": {"role": "assistant", "content": llm_output}
            }]
        }
    }]
)
```

Adapt the variable names (`user_input`, `llm_output`) to match what exists in the code. Extract the user's input and the assistant's response from the existing call and format them into the chat completions structure shown above.

The container creation should run once at startup or module load — not on every request. The registration call goes after each LLM call.

---

## TypeScript / JavaScript integration

### 2a. Install the SDK

```bash
npm install maniac-js
```

If the project uses `yarn` or `pnpm`, use the equivalent command. Add `maniac-js` to `package.json` dependencies.

### 2b. Set the environment variable

Add `MANIAC_API_KEY` to the project's `.env` file (or `.env.example` / `.env.local` if that is the convention in this repo). Use a placeholder value:

```
MANIAC_API_KEY=your-maniac-api-key
```

### 2c. Create a container and register completions

Add the following to the call site you identified. This creates a Maniac container (a task namespace that stores inference logs) and registers completions **after** the existing LLM call. Do not remove or modify the existing call.

Set `CONTAINER_LABEL` to a short, descriptive kebab-case label for the task (e.g. `"support-chat"`, `"code-review"`, `"summarizer"`). Set `INITIAL_MODEL` to the model the existing code is using (e.g. `"openai/gpt-4o"`, `"anthropic/claude-sonnet-4"`). If Maniac does not support that model yet, it will throw an error — in that case, just substitute any supported model as a placeholder (e.g. `"openai/gpt-4o"`).

```typescript
import Maniac from "maniac-js";

const maniac = new Maniac();

// Create the container once
await maniac.containers.create({
  label: "CONTAINER_LABEL",
  initial_model: "INITIAL_MODEL",
});

// --- existing code that calls the LLM ---
// const response = await yourExistingLLMCall(...)
// ---

// Register the completion with Maniac (logging only — does not affect inference)
await maniac.chat.completions.register({
  container: "CONTAINER_LABEL",
  items: [
    {
      input: {
        messages: [{ role: "user", content: userInput }],
      },
      output: {
        choices: [
          {
            message: { role: "assistant", content: llmOutput },
          },
        ],
      },
    },
  ],
});
```

Adapt the variable names (`userInput`, `llmOutput`) to match what exists in the code. Extract the user's input and the assistant's response from the existing call and format them into the chat completions structure shown above.

The container creation should run once at startup or module load — not on every request. The registration call goes after each LLM call.

The `Maniac` client reads `MANIAC_API_KEY` from the environment automatically. You can also pass it explicitly: `new Maniac({ apiKey: "..." })`.

---

## Step 3 — Present the changes

Before making any edits, show the user:

1. Which call site you selected and why
2. The exact code changes you plan to make (as a diff)
3. This reminder:

> **These changes only log completions to Maniac for training data collection. They do not modify, intercept, or affect your existing inference in any way. Your app's behavior, outputs, and costs remain identical.**

Then ask: **"Are these changes OK to implement?"**

Wait for explicit approval before proceeding.

---

## Step 4 — Implement and remind

After the user approves and you have applied the changes, display this message:

> **One last step:** Copy your Maniac API key from your project dashboard at [app.maniac.ai](https://app.maniac.ai) and paste it into your `.env` file as `MANIAC_API_KEY`. If you don't have an account yet, sign up at [app.maniac.ai/auth/register](https://app.maniac.ai/auth/register) and create a project to get your key.
