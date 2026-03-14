<div align="center">

<h1>Claude API — cURL / Raw HTTP</h1>

<p><b>Use raw HTTP requests when no official SDK is available.</b></p>

</div>

<hr>

<p>
Use these examples when the user needs raw HTTP requests or is working in a language without an official SDK.
</p>

<hr>

<h2>Setup</h2>

<pre>
export ANTHROPIC_API_KEY="your-api-key"
</pre>

<hr>

<h2>Basic Message Request</h2>

<pre>
curl https://api.anthropic.com/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "claude-opus-4-6",
    "max_tokens": 1024,
    "messages": [
      {"role": "user", "content": "What is the capital of France?"}
    ]
  }'
</pre>

<hr>

<h2>Streaming (SSE)</h2>

<pre>
curl https://api.anthropic.com/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "claude-opus-4-6",
    "max_tokens": 1024,
    "stream": true,
    "messages": [{"role": "user", "content": "Write a haiku"}]
  }'
</pre>

<p><b>The response is a stream of Server-Sent Events:</b></p>

<pre>
event: message_start
data: {"type":"message_start","message":{"id":"msg_...","type":"message",...}}

event: content_block_start
data: {"type":"content_block_start","index":0,"content_block":{"type":"text","text":""}}

event: content_block_delta
data: {"type":"content_block_delta","index":0,"delta":{"type":"text_delta","text":"Hello"}}

event: content_block_stop
data: {"type":"content_block_stop","index":0}

event: message_delta
data: {"type":"message_delta","delta":{"stop_reason":"end_turn"},"usage":{"output_tokens":12}}

event: message_stop
data: {"type":"message_stop"}
</pre>

<hr>

<h2>Tool Use</h2>

<pre>
curl https://api.anthropic.com/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "claude-opus-4-6",
    "max_tokens": 1024,
    "tools": [{
      "name": "get_weather",
      "description": "Get current weather for a location",
      "input_schema": {
        "type": "object",
        "properties": {
          "location": {"type": "string", "description": "City name"}
        },
        "required": ["location"]
      }
    }],
    "messages": [{"role": "user", "content": "What is the weather in Paris?"}]
  }'
</pre>

<p><b>When Claude responds with a <code>tool_use</code> block, send the result back:</b></p>

<pre>
curl https://api.anthropic.com/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "claude-opus-4-6",
    "max_tokens": 1024,
    "tools": [{
      "name": "get_weather",
      "description": "Get current weather for a location",
      "input_schema": {
        "type": "object",
        "properties": {
          "location": {"type": "string", "description": "City name"}
        },
        "required": ["location"]
      }
    }],
    "messages": [
      {"role": "user", "content": "What is the weather in Paris?"},
      {"role": "assistant", "content": [
        {"type": "text", "text": "Let me check the weather."},
        {"type": "tool_use", "id": "toolu_abc123", "name": "get_weather", "input": {"location": "Paris"}}
      ]},
      {"role": "user", "content": [
        {"type": "tool_result", "tool_use_id": "toolu_abc123", "content": "72°F and sunny"}
      ]}
    ]
  }'
</pre>

<hr>

<h2>Extended Thinking</h2>

<p>
<b>Opus 4.6 and Sonnet 4.6:</b> Use adaptive thinking. <code>budget_tokens</code> is deprecated.
</p>

<p>
<b>Older models:</b> Use <code>"type": "enabled"</code> with <code>"budget_tokens": N</code>.
</p>

<p>
Requirements:
</p>

<ul>
<li>Must be less than <code>max_tokens</code></li>
<li>Minimum value: <b>1024</b></li>
</ul>

<pre>
# Opus 4.6: adaptive thinking (recommended)

curl https://api.anthropic.com/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "claude-opus-4-6",
    "max_tokens": 16000,
    "thinking": {
      "type": "adaptive"
    },
    "output_config": {
      "effort": "high"
    },
    "messages": [{"role": "user", "content": "Solve this step by step..."}]
  }'
</pre>

<hr>

<h2>Required Headers</h2>

<table border="1" cellpadding="8">

<tr>
<th>Header</th>
<th>Value</th>
<th>Description</th>
</tr>

<tr>
<td><code>Content-Type</code></td>
<td>application/json</td>
<td>Required</td>
</tr>

<tr>
<td><code>x-api-key</code></td>
<td>Your API key</td>
<td>Authentication</td>
</tr>

<tr>
<td><code>anthropic-version</code></td>
<td>2023-06-01</td>
<td>API version</td>
</tr>

<tr>
<td><code>anthropic-beta</code></td>
<td>Beta feature IDs</td>
<td>Required for beta features</td>
</tr>

</table>
