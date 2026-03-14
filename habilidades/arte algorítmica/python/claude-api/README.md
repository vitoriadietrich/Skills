<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Claude API — Python</title>

<style>

body{
    font-family: Arial, Helvetica, sans-serif;
    background:#0f172a;
    color:#e5e7eb;
    margin:0;
    padding:40px;
}

h1{
    color:#38bdf8;
}

h2{
    color:#22c55e;
    margin-top:40px;
}

h3{
    color:#facc15;
}

p{
    color:#cbd5f5;
}

pre{
    background:#020617;
    padding:20px;
    border-radius:10px;
    overflow:auto;
    border:1px solid #1e293b;
}

code{
    color:#38bdf8;
}

table{
    width:100%;
    border-collapse:collapse;
    margin-top:20px;
}

th, td{
    border:1px solid #1e293b;
    padding:10px;
}

th{
    background:#020617;
}

hr{
    border:none;
    border-top:1px solid #1e293b;
    margin:40px 0;
}

</style>
</head>

<body>

<h1>Claude API — Python</h1>

<hr>

<h2>Installation</h2>

<pre><code>pip install anthropic</code></pre>

<hr>

<h2>Client Initialization</h2>

<pre><code>
import anthropic

client = anthropic.Anthropic()

client = anthropic.Anthropic(api_key="your-api-key")

async_client = anthropic.AsyncAnthropic()
</code></pre>

<hr>

<h2>Basic Message Request</h2>

<pre><code>
response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "What is the capital of France?"}
    ]
)

print(response.content[0].text)
</code></pre>

<hr>

<h2>System Prompts</h2>

<pre><code>
response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=1024,
    system="You are a helpful coding assistant. Always provide examples in Python.",
    messages=[{"role": "user", "content": "How do I read a JSON file?"}]
)
</code></pre>

<hr>

<h2>Vision (Images)</h2>

<h3>Base64</h3>

<pre><code>
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
</code></pre>

<h3>URL</h3>

<pre><code>
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
                    ""url": "https://example.com/image.png"
"
                }
            },
            {"type": "text", "text": "Describe this image"}
        ]
    }]
)
</code></pre>

<hr>

<h2>Extended Thinking</h2>

<pre><code>
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
</code></pre>

<hr>

<h2>Error Handling</h2>

<pre><code>
import anthropic

try:
    response = client.messages.create(...)
except anthropic.BadRequestError as e:
    print(e.message)
except anthropic.AuthenticationError:
    print("Invalid API key")
except anthropic.RateLimitError:
    print("Rate limited")
</code></pre>

<hr>

<h2>Multi-Turn Conversations</h2>

<pre><code>
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
</code></pre>

<hr>

<h2>Stop Reasons</h2>

<table>

<tr>
<th>Value</th>
<th>Meaning</th>
</tr>

<tr>
<td>end_turn</td>
<td>Claude finished the response</td>
</tr>

<tr>
<td>max_tokens</td>
<td>Reached max_tokens limit</td>
</tr>

<tr>
<td>stop_sequence</td>
<td>Custom stop sequence triggered</td>
</tr>

<tr>
<td>tool_use</td>
<td>Claude wants to call a tool</td>
</tr>

<tr>
<td>pause_turn</td>
<td>Model paused execution</td>
</tr>

<tr>
<td>refusal</td>
<td>Safety refusal</td>
</tr>

</table>

<hr>

<h2>Cost Optimization</h2>

<h3>Choose the Right Model</h3>

<pre><code>
model="claude-opus-4-6"
model="claude-sonnet-4-6"
model="claude-haiku-4-5"
</code></pre>

<hr>

<h2>Token Counting</h2>

<pre><code>
count = client.messages.count_tokens(
    model="claude-opus-4-6",
    messages=messages
)

print(count.input_tokens)
</code></pre>

<hr>

<h2>Retry with Exponential Backoff</h2>

<pre><code>
import time
import random

def retry():

    for attempt in range(5):

        try:
            return client.messages.create(...)

        except Exception:

            delay = 2 ** attempt + random.random()
            time.sleep(delay)
</code></pre>

</body>
</html>
