# Tool Use Concepts — Claude API

This file covers the conceptual foundations of tool use with the Claude API. For language-specific code examples, see the `python/`, `typescript/`, or other language folders.

---

## User-Defined Tools

### Tool Definition Structure

> **Note:** When using the Tool Runner (beta), tool schemas are generated automatically from your function signatures (Python), Zod schemas (TypeScript), annotated classes (Java), `jsonschema` struct tags (Go), or `BaseTool` subclasses (Ruby). The raw JSON schema format below is for the manual approach or SDKs without tool runner support.

Each tool requires a **name**, **description**, and **JSON Schema** for its inputs:

```json
{
  "name": "get_weather",
  "description": "Get current weather for a location",
  "input_schema": {
    "type": "object",
    "properties": {
      "location": {
        "type": "string",
        "description": "City and state, e.g., San Francisco, CA"
      },
      "unit": {
        "type": "string",
        "enum": ["celsius", "fahrenheit"],
        "description": "Temperature unit"
      }
    },
    "required": ["location"]
  }
}
```

**Best practices for tool definitions:**

- Use clear, descriptive names (`get_weather`, `search_database`, `send_email`)  
- Write detailed descriptions — Claude uses these to decide when to use the tool  
- Include descriptions for each property  
- Use `enum` for parameters with fixed values  
- Mark required parameters; make others optional with defaults  

---

## Tool Choice Options

Control when Claude uses tools:

| Value                             | Behavior                                      |
| --------------------------------- | --------------------------------------------- |
| `{"type": "auto"}`                | Claude decides whether to use tools (default) |
| `{"type": "any"}`                 | Claude must use at least one tool             |
| `{"type": "tool", "name": "..."}` | Claude must use the specified tool            |
| `{"type": "none"}`                | Claude cannot use tools                        |

> Any `tool_choice` value can include `"disable_parallel_tool_use": true` to force Claude to use at most one tool per response.

---

## Tool Runner vs Manual Loop

**Tool Runner (Recommended):**  

- SDK handles agentic loop automatically  
- Calls API, detects tool use requests, executes your tools, feeds results back, repeats until done  
- Available in Python, TypeScript, Java, Go, and Ruby SDKs (beta)  
- Python SDK also provides MCP conversion helpers (`anthropic.lib.tools.mcp`)  

**Manual Agentic Loop:**  

- Use when you need fine-grained control (custom logging, conditional execution, human approval)  
- Loop until `stop_reason == "end_turn"`  
- Always append full `response.content`  
- Ensure each `tool_result` includes matching `tool_use_id`  

**Stop reasons for server-side tools:**  

- Default limit of 10 iterations → `stop_reason: "pause_turn"`  
- To continue, re-send user + assistant messages — server resumes automatically  

```python
# Handle pause_turn in your agentic loop
if response.stop_reason == "pause_turn":
    messages = [
        {"role": "user", "content": user_query},
        {"role": "assistant", "content": response.content},
    ]
    response = client.messages.create(
        model="claude-opus-4-6", messages=messages, tools=tools
    )
```

> **Security:** Validate inputs for tools with side effects. Use manual loop if human approval is needed.

---

## Handling Tool Results

1. Execute the tool with the provided input  
2. Send the result back in a `tool_result` message  
3. Continue the conversation  

**Error handling:** Set `"is_error": true` and provide an informative message. Claude may retry or ask for clarification.

**Multiple tool calls:** Handle all requested tools before continuing — send all results in a single `user` message.

---

## Server-Side Tools: Code Execution

- Runs in a secure sandboxed container (1 CPU, 5 GiB RAM, 5 GiB disk)  
- Python 3.11 with data science libraries  
- No internet access  
- Containers persist 30 days and can be reused  

**Tool Declaration:**

```json
{
  "type": "code_execution_20260120",
  "name": "code_execution"
}
```

**Pre-installed Python Libraries:**

- Data science: pandas, numpy, scipy, scikit-learn, statsmodels  
- Visualization: matplotlib, seaborn  
- File processing: openpyxl, xlsxwriter, pillow, pypdf, pdfplumber, python-docx, python-pptx  
- Math: sympy, mpmath  
- Utilities: tqdm, python-dateutil, pytz, sqlite3  

**Supported file types:** CSV, Excel, JSON, XML, images (JPEG, PNG, GIF, WebP), text files (.txt, .md, .py, .js, etc.)

---

## Server-Side Tools: Web Search & Web Fetch

Tool definitions:

```json
[
  { "type": "web_search_20260209", "name": "web_search" },
  { "type": "web_fetch_20260209", "name": "web_fetch" }
]
```

- Support dynamic filtering for Opus 4.6 / Sonnet 4.6  
- Preprocesses results before reaching context window  
- No need to declare `code_execution` unless doing custom computation  

---

## Server-Side Tools: Programmatic Tool Calling

- Claude can call tools directly in code, keeping intermediate results out of the context window  
- Reduces token usage for multi-step workflows  

> Documentation: [Programmatic Tool Calling](https://platform.claude.com/docs/en/agents-and-tools/tool-use/programmatic-tool-calling)

---

## Server-Side Tools: Tool Search

- Dynamically discover relevant tools from large libraries  
- Useful when many tools exist but few are relevant per query  

> Documentation: [Tool Search Tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool)

---

## Server-Side Tools: Computer Use

- Allows Claude to interact with a desktop environment (screenshots, mouse, keyboard)  
- Can be Anthropic-hosted or self-hosted  

> Documentation: [Computer Use Overview](https://platform.claude.com/docs/en/agents-and-tools/computer-use/overview)

---

## Client-Side Tools: Memory

- Stores and retrieves info across conversations  
- Operates on files in `/memories` directory  
- Commands: `view`, `create`, `str_replace`, `insert`, `delete`, `rename`  

> **Security:** Never store secrets or PII. Implement per-user directories and authentication in multi-user systems.  

> Documentation: [Memory Tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool.md)

---

## Structured Outputs

- Constrain responses to a specific JSON schema  
- Features: JSON outputs (`output_config.format`) and strict tool use (`strict: true`)  
- Supported models: Claude Opus 4.6, Sonnet 4.6, Haiku 4.5; legacy models also supported  

**JSON Schema Limitations:**

- Supported: object, array, string, integer, number, boolean, null, `enum`, `const`, `anyOf`, `allOf`, `$ref`/`$def`, string formats (`date-time`, `email`, `uri`, etc.), `additionalProperties: false`  
- Not supported: recursive schemas, numerical/string constraints, complex array constraints  

> Python/TypeScript SDKs automatically handle unsupported constraints client-side.

---

## Tips for Effective Tool Use

1. Provide detailed descriptions  
2. Use specific tool names  
3. Validate inputs before execution  
4. Handle errors gracefully  
5. Limit tool count to avoid confusion  
6. Test tool interactions in various scenarios  

> Documentation: [Claude Tool Use Overview](https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview)
