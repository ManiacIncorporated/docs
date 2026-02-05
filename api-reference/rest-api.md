---
description: >-
  The Maniac API provides openai compatible inference endpoints for interacting
  with both frontier models and your custom models. This reference details the
  available endpoints.
hidden: true
---

# REST API

{% openapi-operation spec="maniac-api" path="/api/v1/chat/completions" method="get" %}
[OpenAPI maniac-api](https://platform.maniac.ai/openapi.json)
{% endopenapi-operation %}

{% openapi-operation spec="maniac-api" path="/api/v1/chat/completions" method="post" %}
[OpenAPI maniac-api](https://platform.maniac.ai/openapi.json)
{% endopenapi-operation %}

{% openapi-operation spec="maniac-api" path="/api/v1/chat/completions/{completion_id}" method="get" %}
[OpenAPI maniac-api](https://platform.maniac.ai/openapi.json)
{% endopenapi-operation %}

{% openapi-operation spec="maniac-api" path="/api/v1/chat/completions/register" method="post" %}
[OpenAPI maniac-api](https://platform.maniac.ai/openapi.json)
{% endopenapi-operation %}

{% openapi-operation spec="maniac-api" path="/api/v1/containers" method="get" %}
[OpenAPI maniac-api](https://platform.maniac.ai/openapi.json)
{% endopenapi-operation %}

{% openapi-operation spec="maniac-api" path="/api/v1/containers" method="post" %}
[OpenAPI maniac-api](https://platform.maniac.ai/openapi.json)
{% endopenapi-operation %}

{% openapi-operation spec="maniac-api" path="/api/v1/containers/{label}" method="get" %}
[OpenAPI maniac-api](https://platform.maniac.ai/openapi.json)
{% endopenapi-operation %}

{% openapi-operation spec="maniac-api" path="/api/v1/containers/{label}" method="delete" %}
[OpenAPI maniac-api](https://platform.maniac.ai/openapi.json)
{% endopenapi-operation %}

{% openapi-operation spec="maniac-api" path="/api/v1/containers/{label}/models" method="post" %}
[OpenAPI maniac-api](https://platform.maniac.ai/openapi.json)
{% endopenapi-operation %}

{% openapi-operation spec="maniac-api" path="/api/v1/files" method="get" %}
[OpenAPI maniac-api](https://platform.maniac.ai/openapi.json)
{% endopenapi-operation %}

{% openapi-operation spec="maniac-api" path="/api/v1/files" method="post" %}
[OpenAPI maniac-api](https://platform.maniac.ai/openapi.json)
{% endopenapi-operation %}

{% openapi-operation spec="maniac-api" path="/api/v1/files/{file_id}" method="get" %}
[OpenAPI maniac-api](https://platform.maniac.ai/openapi.json)
{% endopenapi-operation %}

{% openapi-operation spec="maniac-api" path="/api/v1/files/{file_id}" method="delete" %}
[OpenAPI maniac-api](https://platform.maniac.ai/openapi.json)
{% endopenapi-operation %}

{% openapi-operation spec="maniac-api" path="/api/v1/files/{file_id}/content" method="get" %}
[OpenAPI maniac-api](https://platform.maniac.ai/openapi.json)
{% endopenapi-operation %}

{% openapi-operation spec="maniac-api" path="/api/v1/models" method="get" %}
[OpenAPI maniac-api](https://platform.maniac.ai/openapi.json)
{% endopenapi-operation %}

{% openapi-operation spec="maniac-api" path="/api/v1/evaluators" method="get" %}
[OpenAPI maniac-api](https://platform.maniac.ai/openapi.json)
{% endopenapi-operation %}

{% openapi-operation spec="maniac-api" path="/api/v1/evaluators" method="post" %}
[OpenAPI maniac-api](https://platform.maniac.ai/openapi.json)
{% endopenapi-operation %}

{% openapi-operation spec="maniac-api" path="/v1/evaluation/runs" method="get" %}
[OpenAPI maniac-api](https://platform.maniac.ai/openapi.json)
{% endopenapi-operation %}

{% openapi-operation spec="maniac-api" path="/v1/evaluation/runs" method="post" %}
[OpenAPI maniac-api](https://platform.maniac.ai/openapi.json)
{% endopenapi-operation %}

{% openapi-operation spec="maniac-api" path="/v1/evaluation/runs/{run_id}" method="get" %}
[OpenAPI maniac-api](https://platform.maniac.ai/openapi.json)
{% endopenapi-operation %}

{% openapi-operation spec="maniac-api" path="/v1/evaluation/runs/{run_id}/cancel" method="post" %}
[OpenAPI maniac-api](https://platform.maniac.ai/openapi.json)
{% endopenapi-operation %}

{% openapi-operation spec="maniac-api" path="/api/v1/evaluators/{evaluator_id}" method="get" %}
[OpenAPI maniac-api](https://platform.maniac.ai/openapi.json)
{% endopenapi-operation %}

{% openapi-operation spec="maniac-api" path="/api/v1/evaluators/{evaluator_id}" method="delete" %}
[OpenAPI maniac-api](https://platform.maniac.ai/openapi.json)
{% endopenapi-operation %}

{% openapi-operation spec="maniac-api" path="/api/v1/evaluators/{evaluator_id}" method="patch" %}
[OpenAPI maniac-api](https://platform.maniac.ai/openapi.json)
{% endopenapi-operation %}

{% openapi-operation spec="maniac-api" path="/api/v1/datasets" method="post" %}
[OpenAPI maniac-api](https://platform.maniac.ai/openapi.json)
{% endopenapi-operation %}

{% openapi-operation spec="maniac-api" path="/api/v1/datasets/{dataset_id}" method="get" %}
[OpenAPI maniac-api](https://platform.maniac.ai/openapi.json)
{% endopenapi-operation %}

{% openapi-operation spec="maniac-api" path="/healthz" method="get" %}
[OpenAPI maniac-api](https://platform.maniac.ai/openapi.json)
{% endopenapi-operation %}
