# Message Batches API — Python

O Batches API (`POST /v1/messages/batches`) processa requisições da Messages API de forma **assíncrona** com **50% do custo normal**.

---

## Key Facts

- Até **100.000 requisições** ou **256 MB** por batch
- A maioria dos batches é concluída em **1 hora**; máximo de **24 horas**
- Resultados disponíveis por **29 dias** após a criação
- **50% de redução de custo** em todos os tokens
- Todos os recursos da Messages API suportados (vision, tools, caching, etc.)

---

## Create a Batch

```python
import anthropic
from anthropic.types.message_create_params import MessageCreateParamsNonStreaming
from anthropic.types.messages.batch_create_params import Request

client = anthropic.Anthropic()

message_batch = client.messages.batches.create(
    requests=[
        Request(
            custom_id="request-1",
            params=MessageCreateParamsNonStreaming(
                model="claude-opus-4-6",
                max_tokens=1024,
                messages=[{"role": "user", "content": "Summarize climate change impacts"}]
            )
        ),
        Request(
            custom_id="request-2",
            params=MessageCreateParamsNonStreaming(
                model="claude-opus-4-6",
                max_tokens=1024,
                messages=[{"role": "user", "content": "Explain quantum computing basics"}]
            )
        ),
    ]
)

print(message_batch.id)
print(message_batch.processing_status)
```

---

## Poll for Completion

```python
import time

while True:
    batch = client.messages.batches.retrieve(message_batch.id)

    if batch.processing_status == "ended":
        break

    print(batch.processing_status)
    time.sleep(60)

print("Batch complete")
```

---

## Retrieve Results

```python
for result in client.messages.batches.results(message_batch.id):

    match result.result.type:

        case "succeeded":
            print(result.result.message.content[0].text)

        case "errored":
            print("Request error")

        case "canceled":
            print("Canceled")

        case "expired":
            print("Expired")
```

---

## Cancel a Batch

```python
cancelled = client.messages.batches.cancel(message_batch.id)

print(cancelled.processing_status)
```

---

## Batch with Prompt Caching

```python
shared_system = [
    {"type": "text", "text": "You are a literary analyst."},
    {
        "type": "text",
        "text": large_document_text,
        "cache_control": {"type": "ephemeral"}
    }
]

message_batch = client.messages.batches.create(
    requests=[
        Request(
            custom_id=f"analysis-{i}",
            params=MessageCreateParamsNonStreaming(
                model="claude-opus-4-6",
                max_tokens=1024,
                system=shared_system,
                messages=[{"role": "user", "content": question}]
            )
        )
        for i, question in enumerate(questions)
    ]
)
```

---

## Full End-to-End Example

```python
import anthropic
import time

from anthropic.types.message_create_params import MessageCreateParamsNonStreaming
from anthropic.types.messages.batch_create_params import Request

client = anthropic.Anthropic()

items = [
    "The product quality is excellent!",
    "Terrible customer service.",
    "It's okay."
]

requests = [

    Request(
        custom_id=f"classify-{i}",
        params=MessageCreateParamsNonStreaming(
            model="claude-haiku-4-5",
            max_tokens=50,
            messages=[{
                "role": "user",
                "content": f"Classify as positive/negative/neutral: {text}"
            }]
        )
    )

    for i, text in enumerate(items)
]

batch = client.messages.batches.create(requests=requests)

while True:

    batch = client.messages.batches.retrieve(batch.id)

    if batch.processing_status == "ended":
        break

    time.sleep(10)

results = {}

for result in client.messages.batches.results(batch.id):

    if result.result.type == "succeeded":
        results[result.custom_id] = result.result.message.content[0].text

for custom_id, value in results.items():
    print(custom_id, value)
```
