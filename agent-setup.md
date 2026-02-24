# Maniac Agent Setup

You are instrumenting a codebase to log LLM inference data to **Maniac**, an enterprise AI platform for fine-tuning task-specific models. Follow these instructions precisely.

> **Important:** This integration only *records* LLM completions for training data collection. It does **not** modify, intercept, or affect existing inference in any way. Your application's behavior, latency, and outputs remain completely unchanged.

---

## Step 1 — Scan the codebase

Find all files that make LLM / AI API calls. Look for:

- OpenAI client usage (`openai`, `@ai-sdk/openai`, `OpenAI`)
- Anthropic client usage (`anthropic`, `@ai-sdk/anthropic`, `Anthropic`)
- Google / Gemini usage
- Vercel AI SDK usage (`ai`, `generateText`, `streamText`, `generateObject`)
- Any HTTP calls to `api.openai.com`, `api.anthropic.com`, or similar LLM endpoints
- Other LLM SDKs (Cohere, Mistral, LiteLLM, etc.)

List every call site you find. Then pick the **single most important one** — the call that best represents the core LLM task in this application (e.g. the main chat completion, the primary agent call, the key generation step). You will instrument only this one call site.

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

### 2c. Add the registration call

At the call site you identified, add a Maniac registration call **after** the existing LLM call. Do not remove or modify the existing call. The pattern is:

```python
import os
from maniac import Maniac

maniac = Maniac(api_key=os.getenv("MANIAC_API_KEY"))

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

Adapt the variable names (`user_input`, `llm_output`) to match what exists in the code. If the existing call uses a `messages` array, pass it directly into `input.messages`. If the response is an OpenAI-style object, extract `.choices[0].message.content` for the output.

Set `CONTAINER_LABEL` to a short, descriptive kebab-case label for the task (e.g. `"support-chat"`, `"code-review"`, `"summarizer"`).

---

## TypeScript / JavaScript integration

### 2a. Install the dependency

No Maniac-specific npm package is required. The integration uses the REST API directly.

### 2b. Set the environment variable

Add `MANIAC_API_KEY` to the project's `.env` file (or `.env.example` / `.env.local` if that is the convention in this repo). Use a placeholder value:

```
MANIAC_API_KEY=your-maniac-api-key
```

### 2c. Add the registration call

At the call site you identified, add a Maniac registration call **after** the existing LLM call. Do not remove or modify the existing call.

```typescript
async function registerManiacCompletion({
  container,
  input,
  output,
}: {
  container: string;
  input: { messages: Array<{ role: string; content: string }> };
  output: { choices: Array<{ message: { role: string; content: string } }> };
}) {
  await fetch("https://platform.maniac.ai/api/v1/chat/completions/register", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${process.env.MANIAC_API_KEY}`,
    },
    body: JSON.stringify({ container, items: [{ input, output }] }),
  });
}
```

Then call it after the existing LLM call:

```typescript
// --- existing code that calls the LLM ---
// const response = await yourExistingLLMCall(...)
// ---

// Register the completion with Maniac (logging only — does not affect inference)
await registerManiacCompletion({
  container: "CONTAINER_LABEL",
  input: {
    messages: [{ role: "user", content: userInput }],
  },
  output: {
    choices: [{ message: { role: "assistant", content: llmOutput } }],
  },
});
```

Adapt the variable names (`userInput`, `llmOutput`) to match what exists in the code. If the existing call uses a `messages` array, pass it directly into `input.messages`. If the response is an OpenAI-style object, extract `.choices[0].message.content` for the output.

Set `CONTAINER_LABEL` to a short, descriptive kebab-case label for the task (e.g. `"support-chat"`, `"code-review"`, `"summarizer"`).

#### Vercel AI SDK alternative

If the project uses the Vercel AI SDK, prefer the middleware approach instead of the raw fetch call above. Wrap the model with registration middleware:

```typescript
import { wrapLanguageModel } from "ai";

function maniacRegistrationMiddleware({ container }: { container: string }) {
  return {
    specificationVersion: "v3",
    wrapGenerate: async ({ doGenerate, params }) => {
      const result = await doGenerate();
      const text = result.content
        .filter((c: any) => c.type === "text")
        .map((c: any) => c.text)
        .join("");
      if (text) {
        registerManiacCompletion({
          container,
          input: { messages: params.prompt },
          output: {
            choices: [{ message: { role: "assistant", content: text } }],
          },
        });
      }
      return result;
    },
  };
}

const model = wrapLanguageModel({
  model: yourExistingModel,
  middleware: maniacRegistrationMiddleware({ container: "CONTAINER_LABEL" }),
});
```

---

## Step 3 — Present the changes

Before making any edits, show the user:

1. Which call site you selected and why
2. The exact code changes you plan to make (as a diff)
3. This reminder:

> **These changes only log completions to Maniac for training data collection. They do not modify, intercept, or affect your existing inference in any way. Your app's behavior remains identical.**

Then ask: **"Are these changes OK to implement?"**

Wait for explicit approval before proceeding.

---

## Step 4 — Implement and remind

After the user approves and you have applied the changes, display this message:

> **One last step:** Copy your Maniac API key from your project dashboard at [app.maniac.ai](https://app.maniac.ai) and paste it into your `.env` file as `MANIAC_API_KEY`. If you don't have an account yet, sign up at [app.maniac.ai/auth/register](https://app.maniac.ai/auth/register) and create a project to get your key.
