# Vercel AI SDK

The Maniac platform lets you collect LLM completions for training and evaluation. This guide walks through two ways to integrate Maniac into a project that uses the [Vercel AI SDK](https://sdk.vercel.ai/docs):

* **Option A -- OpenAI-compatible proxy:** Maniac acts as a proxy between your app and the underlying model. Inference and data collection happen in a single call with zero extra wiring.
* **Option B -- Register completions:** You run inference through any provider you like (Anthropic, OpenAI, etc.) and then POST the completion to Maniac after the fact. This is more flexible and works with streaming, custom pipelines, or providers that Maniac does not proxy.

Both options start the same way: you create a **container** on the Maniac platform.

***

### Prerequisites

{% hint style="info" %}
Before you begin, make sure you have:

* A **Maniac API key** (set it as `MANIAC_API_KEY` in your environment).
* **Node.js 18+** and an existing Vercel AI SDK project (or a new one).
* The following packages installed:
  * `ai` (the Vercel AI SDK core)
  * `@ai-sdk/openai-compatible` (needed for Option A)
  * A provider package such as `@ai-sdk/anthropic` or `@ai-sdk/openai` (needed for Option B)
{% endhint %}

***

### Step 1 -- Create a Maniac Container

A **container** is a logical namespace on the Maniac platform that is tied to a specific base model. Every completion you send to that container -- whether proxied or registered -- is stored and can later be used for fine-tuning, evaluation, or analytics.

You only need to create a container once. After that, you reference it by its `label` in all subsequent calls.

```typescript
async function createContainer({ label, initialModel, ...rest }) {
  const headers = {
    "Content-Type": "application/json",
    Authorization: `Bearer ${process.env.MANIAC_API_KEY}`,
  };
  const res = await fetch(`${"https://platform.maniac.ai/api/v1"}/containers`, {
    method: "POST",
    headers,
    body: JSON.stringify({
      label,
      initial_model: initialModel,
      ...rest,
    }),
  });
  const data = await res.json();
  return data;
}
```

{% hint style="warning" %}
The `label` must be unique within your project. The `initialModel` field should match the model you intend to use for inference (e.g. `anthropic/claude-sonnet-4.5`, `openai/gpt-4o-mini`). Maniac uses this to configure the container's proxy.&#x20;
{% endhint %}

***

### Step 2 -- Choose an Integration Path

The sections below cover each option in detail. You can mix and match -- for example, use the proxy in development for convenience and switch to registered completions in production for more control.

***

### Option A: OpenAI-Compatible Proxy

This is the simplest integration path. The Vercel AI SDK's `createOpenAICompatible` helper lets you point your model calls at any OpenAI-compatible endpoint. When you point it at Maniac, every request is forwarded to the underlying model **and** the completion is automatically recorded in your container.

The flow looks like this:

1. Your app calls `generateText` (or `streamText`) with the Maniac provider.
2. Maniac forwards the request to the base model configured on the container.
3. The model response is returned to your app **and** stored on the platform.

```typescript
import { createOpenAICompatible } from "@ai-sdk/openai-compatible";
import { generateText } from "ai";

await createContainer({
  label: "my-container",
  initialModel: "anthropic/claude-sonnet-4.5",
});

const maniac = createOpenAICompatible({
  name: "maniac",
  apiKey: process.env.MANIAC_API_KEY,
  baseURL: "https://platform.maniac.ai/api/v1",
});

const response = await generateText({
  model: maniac("maniac:my-container"),
  prompt: "Hello, world!",
});
```

{% hint style="info" %}
The model string follows the format `maniac:<container-label>`. The `maniac:` prefix tells the platform to route the request to the container you created, which already knows which underlying model to call.
{% endhint %}

***

### Option B: Register Completions

If the OpenAI-compatible proxy does not fit your use case -- for example, you need to call a provider directly, use streaming, or run completions through a custom pipeline -- you can still feed data into a Maniac container by **registering completions** after the fact.

The registration endpoint is:

```
POST https://platform.maniac.ai/api/v1/chat/completions/register
```

Each item you register must include an `input` and an `output`, both formatted according to the [OpenAI Chat Completions](https://platform.openai.com/docs/api-reference/chat) schema. This means:

* **`input`** contains a `messages` array (each message has a `role` and `content`).
* **`output`** contains a `choices` array (each choice has a `message` with `role` and `content`).

Here is a minimal helper function:

```typescript
async function registerCompletion({
  container,
  input,
  output,
}: {
  container: string;
  input: any;
  output: any;
}) {
  fetch(`https://platform.maniac.ai/api/v1/chat/completions/register`, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${process.env.MANIAC_API_KEY}`,
    },
    body: JSON.stringify({ container, items: [{ input, output }] }),
  });
}
```

{% hint style="info" %}
The `items` field is an array, so you can batch multiple completions in a single request if needed.
{% endhint %}

There are several ways to hook this into the Vercel AI SDK. The two most common patterns are **middleware** and **`onFinish` callbacks**.

#### Using Middleware

The Vercel AI SDK supports [language-model middleware](https://sdk.vercel.ai/docs/ai-sdk-core/middleware) -- functions that wrap every call to a model. This is the cleanest approach when you want _every_ completion to be registered automatically, regardless of where in your codebase the call originates.

The middleware below intercepts `generateText` calls via `wrapGenerate`. After the underlying model returns a result, it extracts the text content, formats it into the OpenAI chat-completions schema, and fires off a registration request in the background.

```typescript
export function maniacRegistrationMiddleware({
  container,
}: {
  container: string;
}) {
  return {
    specificationVersion: "v3",
    wrapGenerate: async ({ doGenerate, params }) => {
      const result = await doGenerate();

      const input = { messages: params.prompt };

      const text = result.content
        .filter((c: any) => c.type === "text")
        .map((c: any) => c.text)
        .join("");

      const output = {
        choices: [{ message: { role: "assistant", content: text } }],
      };

      if (text) registerCompletion({ container, input, output });

      return result;
    },
  };
}
```

To use it, wrap your model with `wrapLanguageModel` and pass in the middleware. From that point on, every `generateText` call made with this wrapped model will automatically register completions to your container.

```typescript
import { wrapLanguageModel, generateText } from "ai";
import { anthropic } from "@ai-sdk/anthropic";
import { maniacRegistrationMiddleware } from "./middleware";

const model = wrapLanguageModel({
  model: anthropic("claude-sonnet-4.5"),
  middleware: maniacRegistrationMiddleware({
    container: "my-container",
  }),
});

const { text } = await generateText({
  model,
  prompt: "Hello!",
});

console.log(text);
```

#### Using `onFinish`

If you only need to register completions in one or two places, the `onFinish` callback is a lighter-weight alternative that avoids the overhead of setting up middleware. The callback fires once the model has finished generating, giving you access to the final text.

```typescript
import { generateText } from "ai";
import { anthropic } from "@ai-sdk/anthropic";

prompt = "Hello!";

const { text } = await generateText({
  model: anthropic("claude-sonnet-4.5"),
  prompt,
  onFinish: async ({ text }) => {
    await registerCompletion({
      container: "my-container",
      input: { messages: [{ role: "user", content: prompt }] },
      output: { choices: [{ message: { role: "assistant", content: text } }] },
    });
  },
});
```

***

### Which option should I use?

{% tabs %}
{% tab title="Option A -- Proxy" %}
**Best for:** Quick setup, prototyping, or any situation where you are happy to route inference through Maniac.

* Zero extra code beyond `createOpenAICompatible`.
* Completions are captured automatically.
* You are limited to models and providers that Maniac supports as a proxy.
{% endtab %}

{% tab title="Option B -- Register" %}
**Best for:** Production workloads, custom pipelines, or when you need full control over the inference provider.

* Works with _any_ model or provider.
* Supports streaming via `onFinish` or middleware.
* Requires a small amount of extra wiring (the `registerCompletion` call).
* Can batch-register multiple completions in a single request.
{% endtab %}
{% endtabs %}
