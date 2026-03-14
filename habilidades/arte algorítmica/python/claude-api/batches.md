<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Message Batches API — Python</title>

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

ul{
    margin-top:10px;
}

li{
    margin-bottom:6px;
}

hr{
    border:none;
    border-top:1px solid #1e293b;
    margin:40px 0;
}

</style>
</head>

<body>

<h1>Message Batches API — Python</h1>

<p>
The Batches API (<code>POST /v1/messages/batches</code>) processes Messages API requests asynchronously at <b>50% of standard prices</b>.
</p>

<hr>

<h2>Key Facts</h2>

<ul>
<li>Up to <b>100,000 requests</b> or <b>256 MB</b> per batch</li>
<li>Most batches complete within <b>1 hour</b>; maximum <b>24 hours</b></li>
<li>Results available for <b>29 days</b> after creation</li>
<li><b>50% cost reduction</b> on all token usage</li>
<li>All Messages API features supported (vision, tools, caching, etc.)</li>
</ul>

<hr>

<h2>Create a Batch</h2>

<pre><code>
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
</code></pre>

<hr>

<h2>Poll for Completion</h2>

<pre><code>
import time

while True:
    batch = client.messages.batches.retrieve(message_batch.id)

    if batch.processing_status == "ended":
        break

    print(batch.processing_status)
    time.sleep(60)

print("Batch complete")
</code></pre>

<hr>

<h2>Retrieve Results</h2>

<pre><code>
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
</code></pre>

<hr>

<h2>Cancel a Batch</h2>

<pre><code>
cancelled = client.messages.batches.cancel(message_batch.id)

print(cancelled.processing_status)
</code></pre>

<hr>

<h2>Batch with Prompt Caching</h2>

<pre><code>
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
</code></pre>

<hr>

<h2>Full End-to-End Example</h2>

<pre><code>
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
</code></pre>

</body>
</html>
