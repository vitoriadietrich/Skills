<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Streaming — Python</title>

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

blockquote{
    border-left:4px solid #38bdf8;
    padding-left:15px;
    color:#cbd5f5;
}

</style>
</head>

<body>

<h1>Streaming — Python</h1>

<h2>Quick Start</h2>

<pre><code>
with client.messages.stream(
    model="claude-opus-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Write a story"}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
</code></pre>

<h3>Async</h3>

<pre><code>
async with async_client.messages.stream(
    model="claude-opus-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Write a story"}]
) as stream:
    async for text in stream.text_stream:
        print(text, end="", flush=True)
</code></pre>

<hr>

<h2>Handling Different Content Types</h2>

<p>Claude may return text, thinking blocks, or tool use. Handle each appropriately:</p>

<blockquote>
<strong>Opus 4.6:</strong> Use <code>thinking: {type: "adaptive"}</code>. On older models, use <code>thinking: {type: "enabled", budget_tokens: N}</code> instead.
</blockquote>

<pre><code>
with client.messages.stream(
    model="claude-opus-4-6",
    max_tokens=16000,
    thinking={"type": "adaptive"},
    messages=[{"role": "user", "content": "Analyze this problem"}]
) as stream:
    for event in stream:
        if event.type == "content_block_start":
            if event.content_block.type == "thinking":
                print("\n[Thinking...]")
            elif event.content_block.type == "text":
                print("\n[Response:]")

        elif event.type == "content_block_delta":
            if event.delta.type == "thinking_delta":
                print(event.delta.thinking, end="", flush=True)
            elif event.delta.type == "text_delta":
                print(event.delta.text, end="", flush=True)
</code></pre>

<hr>

<h2>Streaming with Tool Use</h2>

<p>The Python tool runner currently returns complete messages. Use streaming for individual API calls within a manual loop if you need per-token streaming with tools:</p>

<pre><code>
with client.messages.stream(
    model="claude-opus-4-6",
    max_tokens=4096,
    tools=tools,
    messages=messages
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)

    response = stream.get_final_message()
    # Continue with tool execution if response.stop_reason == "tool_use"
</code></pre>

<hr>

<h2>Getting the Final Message</h2>

<pre><code>
with client.messages.stream(
    model="claude-opus-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello"}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)

    # Get full message after streaming
    final_message = stream.get_final_message()
    print(f"\n\nTokens used: {final_message.usage.output_tokens}")
</code></pre>

<hr>

<h2>Streaming with Progress Updates</h2>

<pre><code>
def stream_with_progress(client, **kwargs):
    """Stream a response with progress updates."""
    total_tokens = 0
    content_parts = []

    with client.messages.stream(**kwargs) as stream:
        for event in stream:
            if event.type == "content_block_delta":
                if event.delta.type == "text_delta":
                    text = event.delta.text
                    content_parts.append(text)
                    print(text, end="", flush=True)

            elif event.type == "message_delta":
                if event.usage and event.usage.output_tokens is not None:
                    total_tokens = event.usage.output_tokens

        final_message = stream.get_final_message()

    print(f"\n\n[Tokens used: {total_tokens}]")
    return "".join(content_parts)
</code></pre>

<hr>

<h2>Error Handling in Streams</h2>

<pre><code>
try:
    with client.messages.stream(
        model="claude-opus-4-6",
        max_tokens=1024,
        messages=[{"role": "user", "content": "Write a story"}]
    ) as stream:
        for text in stream.text_stream:
            print(text, end="", flush=True)
except anthropic.APIConnectionError:
    print("\nConnection lost. Please retry.")
except anthropic.RateLimitError:
    print("\nRate limited. Please wait and retry.")
except anthropic.APIStatusError as e:
    print(f"\nAPI error: {e.status_code}")
</code></pre>

<hr>

<h2>Stream Event Types</h2>

<table>

<tr>
<th>Event Type</th>
<th>Description</th>
<th>When it fires</th>
</tr>

<tr>
<td>message_start</td>
<td>Contains message metadata</td>
<td>Once at the beginning</td>
</tr>

<tr>
<td>content_block_start</td>
<td>New content block beginning</td>
<td>When a text/tool_use block starts</td>
</tr>

<tr>
<td>content_block_delta</td>
<td>Incremental content update</td>
<td>For each token/chunk</td>
</tr>

<tr>
<td>content_block_stop</td>
<td>Content block complete</td>
<td>When a block finishes</td>
</tr>

<tr>
<td>message_delta</td>
<td>Message-level updates</td>
<td>Contains stop_reason, usage</td>
</tr>

<tr>
<td>message_stop</td>
<td>Message complete</td>
<td>Once at the end</td>
</tr>

</table>

<hr>

<h2>Best Practices</h2>

<ol>
<li><strong>Always flush output</strong> — Use <code>flush=True</code> to show tokens immediately</li>
<li><strong>Handle partial responses</strong> — If the stream is interrupted, you may have incomplete content</li>
<li><strong>Track token usage</strong> — The <code>message_delta</code> event contains usage information</li>
<li><strong>Use timeouts</strong> — Set appropriate timeouts for your application</li>
<li><strong>Default to streaming</strong> — Use <code>.get_final_message()</code> to get the complete response even when streaming, giving you timeout protection without needing to handle individual events</li>
</ol>

</body>
</html>
