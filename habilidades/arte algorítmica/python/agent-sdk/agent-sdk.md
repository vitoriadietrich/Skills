<div align="center">

<h1>Claude Agent SDK — Python</h1>

<p><b>High-level SDK for building AI agents with Claude</b></p>

</div>

<hr>

<p>
The Claude Agent SDK provides a higher-level interface for building AI agents with built-in tools, safety features, and agentic capabilities.
</p>

<hr>

<h2>Installation</h2>

<pre>
pip install claude-agent-sdk
</pre>

<hr>

<h2>Quick Start</h2>

<pre>
import anyio
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

async def main():
    async for message in query(
        prompt="Explain this codebase",
        options=ClaudeAgentOptions(allowed_tools=["Read", "Glob", "Grep"])
    ):
        if isinstance(message, ResultMessage):
            print(message.result)

anyio.run(main)
</pre>

<hr>

<h2>Built-in Tools</h2>

<table border="1" cellpadding="8">

<tr>
<th>Tool</th>
<th>Description</th>
</tr>

<tr>
<td>Read</td>
<td>Read files in the workspace</td>
</tr>

<tr>
<td>Write</td>
<td>Create new files</td>
</tr>

<tr>
<td>Edit</td>
<td>Make precise edits to existing files</td>
</tr>

<tr>
<td>Bash</td>
<td>Execute shell commands</td>
</tr>

<tr>
<td>Glob</td>
<td>Find files by pattern</td>
</tr>

<tr>
<td>Grep</td>
<td>Search files by content</td>
</tr>

<tr>
<td>WebSearch</td>
<td>Search the web for information</td>
</tr>

<tr>
<td>WebFetch</td>
<td>Fetch and analyze web pages</td>
</tr>

<tr>
<td>AskUserQuestion</td>
<td>Ask user clarifying questions</td>
</tr>

<tr>
<td>Agent</td>
<td>Spawn subagents</td>
</tr>

</table>

<hr>

<h2>Primary Interfaces</h2>

<h3>query() — Simple One-Shot Usage</h3>

<p>
The <code>query()</code> function is the simplest way to run an agent.
It returns an async iterator of messages.
</p>

<pre>
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

async for message in query(
    prompt="Explain this codebase",
    options=ClaudeAgentOptions(allowed_tools=["Read", "Glob", "Grep"])
):
    if isinstance(message, ResultMessage):
        print(message.result)
</pre>

<hr>

<h3>ClaudeSDKClient — Full Control</h3>

<p>
ClaudeSDKClient provides full control over the agent lifecycle.
Use it when you need custom tools, hooks, streaming, or the ability to interrupt execution.
</p>

<pre>
import anyio
from claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions, AssistantMessage, TextBlock

async def main():
    options = ClaudeAgentOptions(allowed_tools=["Read", "Glob", "Grep"])

    async with ClaudeSDKClient(options=options) as client:
        await client.query("Explain this codebase")

        async for message in client.receive_response():
            if isinstance(message, AssistantMessage):
                for block in message.content:
                    if isinstance(block, TextBlock):
                        print(block.text)

anyio.run(main)
</pre>

<p>ClaudeSDKClient supports:</p>

<ul>

<li>Async context manager (<code>async with</code>)</li>
<li><code>client.query(prompt)</code> to send prompts</li>
<li><code>receive_response()</code> for streaming responses</li>
<li><code>interrupt()</code> to stop execution</li>
<li>Custom tools via MCP servers</li>

</ul>

<hr>

<h2>Permission System</h2>

<pre>
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

async for message in query(
    prompt="Refactor the authentication module",
    options=ClaudeAgentOptions(
        allowed_tools=["Read", "Edit", "Write"],
        permission_mode="acceptEdits"
    )
):
    if isinstance(message, ResultMessage):
        print(message.result)
</pre>

<h3>Permission Modes</h3>

<ul>

<li><b>default</b> — Prompt for dangerous operations</li>
<li><b>plan</b> — Planning only</li>
<li><b>acceptEdits</b> — Auto accept edits</li>
<li><b>dontAsk</b> — No prompts (CI/CD)</li>
<li><b>bypassPermissions</b> — Skip prompts entirely (requires allow_dangerously_skip_permissions=True)</li>

</ul>

<hr>

<h2>MCP (Model Context Protocol) Support</h2>

<pre>
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

async for message in query(
    prompt="Open example.com and describe what you see",
    options=ClaudeAgentOptions(
        mcp_servers={
            "playwright": {"command": "npx", "args": ["@playwright/mcp@latest"]}
        }
    )
):
    if isinstance(message, ResultMessage):
        print(message.result)
</pre>

<hr>

<h2>Hooks</h2>

<p>
Hooks allow you to customize agent behavior.
</p>

<pre>
from claude_agent_sdk import query, ClaudeAgentOptions, HookMatcher, ResultMessage

async def log_file_change(input_data, tool_use_id, context):
    file_path = input_data.get('tool_input', {}).get('file_path', 'unknown')
    print(f"Modified: {file_path}")
    return {}

async for message in query(
    prompt="Refactor utils.py",
    options=ClaudeAgentOptions(
        permission_mode="acceptEdits",
        hooks={
            "PostToolUse": [HookMatcher(matcher="Edit|Write", hooks=[log_file_change])]
        }
    )
):
    if isinstance(message, ResultMessage):
        print(message.result)
</pre>

<h3>Hook Events</h3>

<ul>

<li>PreToolUse</li>
<li>PostToolUse</li>
<li>PostToolUseFailure</li>
<li>Notification</li>
<li>UserPromptSubmit</li>
<li>SessionStart</li>
<li>SessionEnd</li>
<li>Stop</li>
<li>SubagentStart</li>
<li>SubagentStop</li>
<li>PreCompact</li>
<li>PermissionRequest</li>
<li>Setup</li>
<li>TeammateIdle</li>
<li>TaskCompleted</li>
<li>ConfigChange</li>

</ul>

<hr>

<h2>Common Options</h2>

<p>
<code>query()</code> takes a prompt and an options object (<code>ClaudeAgentOptions</code>).
</p>

<table border="1" cellpadding="8">

<tr>
<th>Option</th>
<th>Type</th>
<th>Description</th>
</tr>

<tr>
<td>cwd</td>
<td>string</td>
<td>Working directory</td>
</tr>

<tr>
<td>allowed_tools</td>
<td>list</td>
<td>Allowed tools</td>
</tr>

<tr>
<td>tools</td>
<td>list</td>
<td>Built-in tools to enable</td>
</tr>

<tr>
<td>disallowed_tools</td>
<td>list</td>
<td>Tools to disable</td>
</tr>

<tr>
<td>permission_mode</td>
<td>string</td>
<td>Permission handling</td>
</tr>

<tr>
<td>allow_dangerously_skip_permissions</td>
<td>bool</td>
<td>Required for bypassPermissions</td>
</tr>

<tr>
<td>mcp_servers</td>
<td>dict</td>
<td>MCP server integrations</td>
</tr>

<tr>
<td>hooks</td>
<td>dict</td>
<td>Agent hooks</td>
</tr>

<tr>
<td>system_prompt</td>
<td>string</td>
<td>Custom system prompt</td>
</tr>

<tr>
<td>max_turns</td>
<td>int</td>
<td>Maximum agent turns</td>
</tr>

<tr>
<td>max_budget_usd</td>
<td>float</td>
<td>Budget limit</td>
</tr>

<tr>
<td>model</td>
<td>string</td>
<td>Model ID</td>
</tr>

<tr>
<td>agents</td>
<td>dict</td>
<td>Subagent definitions</td>
</tr>

<tr>
<td>output_format</td>
<td>dict</td>
<td>Structured output schema</td>
</tr>

<tr>
<td>thinking</td>
<td>dict</td>
<td>Reasoning control</td>
</tr>

<tr>
<td>betas</td>
<td>list</td>
<td>Enable beta features</td>
</tr>

<tr>
<td>setting_sources</td>
<td>list</td>
<td>Settings sources</td>
</tr>

<tr>
<td>env</td>
<td>dict</td>
<td>Environment variables</td>
</tr>

</table>

<hr>

<h2>Message Types</h2>

<pre>
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage, SystemMessage

async for message in query(
    prompt="Find TODO comments",
    options=ClaudeAgentOptions(allowed_tools=["Read", "Glob", "Grep"])
):
    if isinstance(message, ResultMessage):
        print(message.result)

    elif isinstance(message, SystemMessage) and message.subtype == "init":
        session_id = message.session_id
</pre>

<hr>

<h2>Subagents</h2>

<pre>
from claude_agent_sdk import query, ClaudeAgentOptions, AgentDefinition, ResultMessage

async for message in query(
    prompt="Use the code-reviewer agent to review this codebase",
    options=ClaudeAgentOptions(
        allowed_tools=["Read", "Glob", "Grep", "Agent"],
        agents={
            "code-reviewer": AgentDefinition(
                description="Expert code reviewer for quality and security reviews.",
                prompt="Analyze code quality and suggest improvements.",
                tools=["Read", "Glob", "Grep"]
            )
        }
    )
):
    if isinstance(message, ResultMessage):
        print(message.result)
</pre>

<hr>

<h2>Error Handling</h2>

<pre>
from claude_agent_sdk import query, ClaudeAgentOptions, CLINotFoundError, CLIConnectionError, ResultMessage

try:
    async for message in query(
        prompt="...",
        options=ClaudeAgentOptions(allowed_tools=["Read"])
    ):
        if isinstance(message, ResultMessage):
            print(message.result)

except CLINotFoundError:
    print("Claude Code CLI not found. Install with: pip install claude-agent-sdk")

except CLIConnectionError as e:
    print(f"Connection error: {e}")
</pre>

<hr>

<h2>Best Practices</h2>

<ul>

<li>Always specify <b>allowed_tools</b></li>

<li>Set a working directory (<b>cwd</b>)</li>

<li>Use appropriate permission modes</li>

<li>Handle all message types</li>

<li>Limit <b>max_turns</b> to avoid runaway agents</li>

</ul>
