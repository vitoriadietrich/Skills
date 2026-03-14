# Claude API — Python

---

## Installation

```bash
pip install anthropic
```

---

## Client Initialization

```python
import anthropic

# Default (uses ANTHROPIC_API_KEY env var)
client = anthropic.Anthropic()

# Explicit API key
client = anthropic.Anthropic(api_key="your-api-key")

# Async client
async_client = anthropic.AsyncAnthropic()
```

---

## Basic Message Request

```python
response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "What is the capital of France?"}
    ]
)

print(response.content[0].text)
```

---

## System Prompts

```python
response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=1024,
    system="You are a helpful coding assistant. Always provide examples in Python.",
    messages=[{"role": "user", "content": "How do I read a JSON file?"}]
)
```

---

## Vision (Images)

### Base64

```python
import base64

with open("image.png", "rb") as f:
    image_data = base64.standard_b64encode(f.read()).decode("utf-8")

response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": [
            {
                "type": "image",
                "source": {
                    "type": "base64",
                    "media_type": "image/png",
                    "data": image_data
                }
            },
            {"type": "text", "text": "What's in this image?"}
        ]
    }]
)
```

### URL

```python
response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": [
            {
                "type": "image",
                "source": {
                    "type": "url",
                    "url": "https://example.com/image.png"
                }
            },
            {"type": "text", "text": "Describe this image"}
        ]
    }]
)
```

---

## Extended Thinking

```python
response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=16000,
    thinking={"type": "adaptive"},
    output_config={"effort": "high"},
    messages=[{"role": "user", "content": "Solve this step by step"}]
)

for block in response.content:
    if block.type == "thinking":
        print(block.thinking)
    elif block.type == "text":
        print(block.text)
```

---

## Error Handling

```python
import anthropic

try:
    response = client.messages.create(...)
except anthropic.BadRequestError as e:
    print(e.message)
except anthropic.AuthenticationError:
    print("Invalid API key")
except anthropic.RateLimitError:
    print("Rate limited")
```

---

## Multi-Turn Conversations

```python
class ConversationManager:

    def __init__(self, client, model, system=None):
        self.client = client
        self.model = model
        self.system = system
        self.messages = []

    def send(self, message):

        self.messages.append({
            "role": "user",
            "content": message
        })

        response = self.client.messages.create(
            model=self.model,
            messages=self.messages,
            max_tokens=1024
        )

        text = response.content[0].text

        self.messages.append({
            "role": "assistant",
            "content": text
        })

        return text
```

---

## Stop Reasons

| Value        | Meaning                        |
| ------------ | ------------------------------ |
| end_turn     | Claude finished the response   |
| max_tokens   | Reached max_tokens limit       |
| stop_sequence| Custom stop sequence triggered |
| tool_use     | Claude wants to call a tool    |
| pause_turn   | Model paused execution         |
| refusal      | Safety refusal                 |

---

## Cost Optimization

### Choose the Right Model

```python
model="claude-opus-4-6"
model="claude-sonnet-4-6"
model="claude-haiku-4-5"
```

---

## Token Counting

```python
count = client.messages.count_tokens(
    model="claude-opus-4-6",
    messages=messages
)

print(count.input_tokens)
```

---

## Retry with Exponential Backoff

```python
import time
import random

def retry():

    for attempt in range(5):

        try:
            return client.messages.create(...)

        except Exception:

            delay = 2 ** attempt + random.random()
            time.sleep(delay)
```
