# Tool Use — Python

Para uma visão conceitual sobre definição de ferramentas, escolha de ferramentas e dicas, veja `shared/tool-use-concepts.md`.

---

## Tool Runner (Recommended)

**Beta:** O tool runner está em beta no SDK Python.

Use o decorator `@beta_tool` para definir ferramentas como funções tipadas e passe-as para `client.beta.messages.tool_runner()`:

```python
import anthropic
from anthropic import beta_tool

client = anthropic.Anthropic()

@beta_tool
def get_weather(location: str, unit: str = "celsius") -> str:
    """Get current weather for a location."""
    return f"72°F and sunny in {location}"

runner = client.beta.messages.tool_runner(
    model="claude-opus-4-6",
    max_tokens=4096,
    tools=[get_weather],
    messages=[{"role": "user", "content": "What's the weather in Paris?"}],
)

for message in runner:
    print(message)
```

Para uso assíncrono, use `@beta_async_tool` com funções `async def`.

---

## MCP Tool Conversion Helpers

**Beta:** Converta ferramentas MCP (Model Context Protocol), prompts e recursos para tipos da Anthropic API.  
Requer `pip install anthropic[mcp]` (Python 3.10+).

> Nota: A API Claude também suporta o parâmetro `mcp_servers` para conectar-se diretamente a servidores MCP remotos.

### MCP Tools com Tool Runner

```python
from anthropic import AsyncAnthropic
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
```

---

## Manual Agentic Loop

```python
import anthropic

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
```

---

## Handling Tool Results

```python
response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=1024,
    tools=tools,
    messages=[{"role": "user", "content": "What's the weather in Paris?"}]
)
```

---

## Multiple Tool Calls

```python
tool_results = []

for block in response.content:
    if block.type == "tool_use":
        result = execute_tool(block.name, block.input)
        tool_results.append({
            "type": "tool_result",
            "tool_use_id": block.id,
            "content": result
        })
```

---

## Error Handling in Tool Results

```python
tool_result = {
    "type": "tool_result",
    "tool_use_id": tool_use_id,
    "content": "Error: Location 'xyz' not found. Please provide a valid city name.",
    "is_error": True
}
```

---

## Tool Choice

```python
tool_choice={"type": "tool", "name": "get_weather"}
```

---

## Code Execution

### Basic Usage

```python
import anthropic

client = anthropic.Anthropic()

response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=4096,
    messages=[{"role": "user", "content": "Calculate the mean and standard deviation"}],
    tools=[{"type": "code_execution_20260120", "name": "code_execution"}]
)
```

---

## Memory Tool

### Basic Usage

```python
import anthropic

client = anthropic.Anthropic()

response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=2048,
    messages=[{"role": "user", "content": "Remember that my preferred language is Python."}],
    tools=[{"type": "memory_20250818", "name": "memory"}],
)
```

---

## Structured Outputs

### JSON Outputs (Pydantic — Recommended)

```python
from pydantic import BaseModel
from typing import List
import anthropic

class ContactInfo(BaseModel):
    name: str
    email: str
    plan: str
    interests: List[str]
    demo_requested: bool
```

### Raw Schema

```python
output_config={
 "format":{
  "type":"json_schema"
 }
}
```

### Strict Tool Use

```python
tools=[{"name":"book_flight","strict":True}]
```

### Using Both Together

```python
response = client.messages.create(
 model="claude-opus-4-6",
 max_tokens=1024
)
```
