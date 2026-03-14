<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Agent SDK Patterns — Python</title>

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

hr{
    border:none;
    border-top:1px solid #1e293b;
    margin:40px 0;
}

</style>
</head>

<body>

<h1>Agent SDK Patterns — Python</h1>

<hr>

<h2>Basic Agent</h2>

<pre><code>
import anyio
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

async def main():
    async for message in query(
        prompt="Explain what this repository does",
        options=ClaudeAgentOptions(
            cwd="/path/to/project",
            allowed_tools=["Read", "Glob", "Grep"]
        )
    ):
        if isinstance(message, ResultMessage):
            print(message.result)

anyio.run(main)
</code></pre>

<hr>

<h2>Custom Tools</h2>

<p>Custom tools require an MCP server.</p>

<pre><code>
import anyio
from claude_agent_sdk import (
    tool,
    create_sdk_mcp_server,
    ClaudeSDKClient,
    ClaudeAgentOptions,
    AssistantMessage,
    TextBlock,
)

@tool("get_weather", "Get the current weather for a location", {"location": str})
async def get_weather(args):
    location = args["location"]
    return {"content": [{"type": "text", "text": f"The weather in {location} is sunny and 72°F."}]}

server = create_sdk_mcp_server("weather-tools", tools=[get_weather])

async def main():
    options = ClaudeAgentOptions(mcp_servers={"weather": server})
    async with ClaudeSDKClient(options=options) as client:
        await client.query("What's the weather in Paris?")
        async for message in client.receive_response():
            if isinstance(message, AssistantMessage):
                for block in message.content:
                    if isinstance(block, TextBlock):
                        print(block.text)

anyio.run(main)
</code></pre>

<hr>

<h2>Hooks</h2>

<h3>After Tool Use Hook</h3>

<pre><code>
import anyio
from datetime import datetime
from claude_agent_sdk import query, ClaudeAgentOptions, HookMatcher, ResultMessage

async def log_file_change(input_data, tool_use_id, context):
    file_path = input_data.get('tool_input', {}).get('file_path', 'unknown')
    with open('./audit.log', 'a') as f:
        f.write(f"{datetime.now()}: modified {file_path}\n")
    return {}

async def main():
    async for message in query(
        prompt="Refactor utils.py to improve readability",
        options=ClaudeAgentOptions(
            allowed_tools=["Read", "Edit", "Write"],
            permission_mode="acceptEdits",
            hooks={
                "PostToolUse": [HookMatcher(matcher="Edit|Write", hooks=[log_file_change])]
            }
        )
    ):
        if isinstance(message, ResultMessage):
            print(message.result)

anyio.run(main)
</code></pre>

<hr>

<h2>Subagents</h2>

<pre><code>
import anyio
from claude_agent_sdk import query, ClaudeAgentOptions, AgentDefinition, ResultMessage

async def main():
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

anyio.run(main)
</code></pre>

<hr>

<h2>MCP Server Integration</h2>

<h3>Browser Automation (Playwright)</h3>

<pre><code>
import anyio
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

async def main():
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

anyio.run(main)
</code></pre>

<h3>Database Access (PostgreSQL)</h3>

<pre><code>
import os
import anyio
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

async def main():
    async for message in query(
        prompt="Show me the top 10 users by order count",
        options=ClaudeAgentOptions(
            mcp_servers={
                "postgres": {
                    "command": "npx",
                    "args": ["-y", "@modelcontextprotocol/server-postgres"],
                    "env": {"DATABASE_URL": os.environ["DATABASE_URL"]}
                }
            }
        )
    ):
        if isinstance(message, ResultMessage):
            print(message.result)

anyio.run(main)
</code></pre>

<hr>

<h2>Permission Modes</h2>

<pre><code>
import anyio
from claude_agent_sdk import query, ClaudeAgentOptions

async def main():

    async for message in query(
        prompt="Delete all test files",
        options=ClaudeAgentOptions(
            allowed_tools=["Bash"],
            permission_mode="default"
        )
    ):
        pass

    async for message in query(
        prompt="Refactor the auth system",
        options=ClaudeAgentOptions(
            allowed_tools=["Read", "Edit"],
            permission_mode="plan"
        )
    ):
        pass

    async for message in query(
        prompt="Refactor this module",
        options=ClaudeAgentOptions(
            allowed_tools=["Read", "Edit"],
            permission_mode="acceptEdits"
        )
    ):
        pass

    async for message in query(
        prompt="Set up the development environment",
        options=ClaudeAgentOptions(
            allowed_tools=["Bash", "Write"],
            permission_mode="bypassPermissions",
            allow_dangerously_skip_permissions=True
        )
    ):
        pass

anyio.run(main)
</code></pre>

<hr>

<h2>Error Recovery</h2>

<pre><code>
import anyio
from claude_agent_sdk import (
    query,
    ClaudeAgentOptions,
    CLINotFoundError,
    CLIConnectionError,
    ProcessError,
    ResultMessage,
)

async def run_with_recovery():
    try:
        async for message in query(
            prompt="Fix the failing tests",
            options=ClaudeAgentOptions(
                allowed_tools=["Read", "Edit", "Bash"],
                max_turns=10
            )
        ):
            if isinstance(message, ResultMessage):
                print(message.result)
    except CLINotFoundError:
        print("Claude Code CLI not found. Install with: pip install claude-agent-sdk")
    except CLIConnectionError as e:
        print(f"Connection error: {e}")
    except ProcessError as e:
        print(f"Process error: {e}")

anyio.run(run_with_recovery)
</code></pre>

<hr>

<h2>Session Resumption</h2>

<pre><code>
import anyio
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage, SystemMessage

async def main():
    session_id = None

    async for message in query(
        prompt="Read the authentication module",
        options=ClaudeAgentOptions(allowed_tools=["Read", "Glob"])
    ):
        if isinstance(message, SystemMessage) and message.subtype == "init":
            session_id = message.session_id

    async for message in query(
        prompt="Now find all places that call it",
        options=ClaudeAgentOptions(resume=session_id)
    ):
        if isinstance(message, ResultMessage):
            print(message.result)

anyio.run(main)
</code></pre>

<hr>

<h2>Custom System Prompt</h2>

<pre><code>
import anyio
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

async def main():
    async for message in query(
        prompt="Review this code",
        options=ClaudeAgentOptions(
            allowed_tools=["Read", "Glob", "Grep"],
            system_prompt="""You are a senior code reviewer focused on:
1. Security vulnerabilities
2. Performance issues
3. Code maintainability

Always provide specific line numbers and suggestions for improvement."""
        )
    ):
        if isinstance(message, ResultMessage):
            print(message.result)

anyio.run(main)
</code></pre>

</body>
</html>
