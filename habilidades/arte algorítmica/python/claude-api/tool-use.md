<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Tool Use — Python</title>

<style>
body{
 font-family: Arial, Helvetica, sans-serif;
 background:#0f172a;
 color:#e5e7eb;
 margin:0;
 padding:40px;
}

h1{ color:#38bdf8; }
h2{ color:#22c55e; margin-top:40px; }
h3{ color:#facc15; }

p{ color:#cbd5f5; }

pre{
 background:#020617;
 padding:20px;
 border-radius:10px;
 overflow:auto;
 border:1px solid #1e293b;
}

code{ color:#38bdf8; }

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

<h1>Tool Use — Python</h1>

<p>For conceptual overview (tool definitions, tool choice, tips), see shared/tool-use-concepts.md.</p>

<h2>Tool Runner (Recommended)</h2>

<p><strong>Beta:</strong> The tool runner is in beta in the Python SDK.</p>

<p>Use the <code>@beta_tool</code> decorator to define tools as typed functions, then pass them to <code>client.beta.messages.tool_runner()</code>:</p>

<pre><code>import anthropic
from anthropic import beta_tool

client = anthropic.Anthropic()

@beta_tool
def get_weather(location: str, unit: str = "celsius") -> str:
    """Get current weather for a location.

    Args:
        location: City and state, e.g., San Francisco, CA.
        unit: Temperature unit, either "celsius" or "fahrenheit".
    """
    # Your implementation here
    return f"72°F and sunny in {location}"

# The tool runner handles the agentic loop automatically
runner = client.beta.messages.tool_runner(
    model="claude-opus-4-6",
    max_tokens=4096,
    tools=[get_weather],
    messages=[{"role": "user", "content": "What's the weather in Paris?"}],
)

# Each iteration yields a BetaMessage; iteration stops when Claude is done
for message in runner:
    print(message)
</code></pre>

<p>For async usage, use <code>@beta_async_tool</code> with <code>async def</code> functions.</p>

<hr>

<h2>MCP Tool Conversion Helpers</h2>

<p><strong>Beta.</strong> Convert MCP (Model Context Protocol) tools, prompts, and resources to Anthropic API types for use with the tool runner. Requires <code>pip install anthropic[mcp]</code> (Python 3.10+).</p>

<blockquote>
Note: The Claude API also supports an <code>mcp_servers</code> parameter that lets Claude connect directly to remote MCP servers.
</blockquote>

<h3>MCP Tools with Tool Runner</h3>

<pre><code>from anthropic import AsyncAnthropic
from anthropic.lib.tools.mcp import async_mcp_tool
from mcp import ClientSession
from mcp.client.stdio import stdio_client, StdioServerParameters

client = AsyncAnthropic()

async with stdio_client(StdioServerParameters(command="mcp-server")) as (read, write):
    async with ClientSession(read, write) as mcp_client:
        await mcp_client.initialize()

        tools_result = await mcp_client.list_tools()
        runner = await client.beta.messages.tool_runner(
            model="claude-opus-4-6",
            max_tokens=1024,
            messages=[{"role": "user", "content": "Use the available tools"}],
            tools=[async_mcp_tool(t, mcp_client) for t in tools_result.tools],
        )
        async for message in runner:
            print(message)
</code></pre>

<hr>

<h2>Manual Agentic Loop</h2>

<pre><code>import anthropic

client = anthropic.Anthropic()
tools = [...]
messages = [{"role": "user", "content": user_input}]

while True:
    response = client.messages.create(
        model="claude-opus-4-6",
        max_tokens=4096,
        tools=tools,
        messages=messages
    )

    if response.stop_reason == "end_turn":
        break
</code></pre>

<hr>

<h2>Handling Tool Results</h2>

<pre><code>response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=1024,
    tools=tools,
    messages=[{"role": "user", "content": "What's the weather in Paris?"}]
)
</code></pre>

<hr>

<h2>Multiple Tool Calls</h2>

<pre><code>tool_results = []

for block in response.content:
    if block.type == "tool_use":
        result = execute_tool(block.name, block.input)
        tool_results.append({
            "type": "tool_result",
            "tool_use_id": block.id,
            "content": result
        })
</code></pre>

<hr>

<h2>Error Handling in Tool Results</h2>

<pre><code>tool_result = {
    "type": "tool_result",
    "tool_use_id": tool_use_id,
    "content": "Error: Location 'xyz' not found. Please provide a valid city name.",
    "is_error": True
}
</code></pre>

<hr>

<h2>Tool Choice</h2>

<pre><code>tool_choice={"type": "tool", "name": "get_weather"}
</code></pre>

<hr>

<h2>Code Execution</h2>

<h3>Basic Usage</h3>

<pre><code>import anthropic

client = anthropic.Anthropic()

response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=4096,
    messages=[{
        "role": "user",
        "content": "Calculate the mean and standard deviation"
    }],
    tools=[{
        "type": "code_execution_20260120",
        "name": "code_execution"
    }]
)
</code></pre>

<hr>

<h2>Memory Tool</h2>

<h3>Basic Usage</h3>

<pre><code>import anthropic

client = anthropic.Anthropic()

response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=2048,
    messages=[{"role": "user", "content": "Remember that my preferred language is Python."}],
    tools=[{"type": "memory_20250818", "name": "memory"}],
)
</code></pre>

<hr>

<h2>Structured Outputs</h2>

<h3>JSON Outputs (Pydantic — Recommended)</h3>

<pre><code>from pydantic import BaseModel
from typing import List
import anthropic

class ContactInfo(BaseModel):
    name: str
    email: str
    plan: str
    interests: List[str]
    demo_requested: bool
</code></pre>

<h3>Raw Schema</h3>

<pre><code>output_config={
 "format":{
  "type":"json_schema"
 }
}
</code></pre>

<h3>Strict Tool Use</h3>

<pre><code>tools=[{
 "name":"book_flight",
 "strict":True
}]
</code></pre>

<h3>Using Both Together</h3>

<pre><code>response = client.messages.create(
 model="claude-opus-4-6",
 max_tokens=1024
)
</code></pre>

</body>
</html>
