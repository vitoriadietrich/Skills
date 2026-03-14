<div align="center">

<h1>Claude API — C#</h1>

<p><b>Official Anthropic SDK for C#</b></p>

</div>

<hr>

<p>
<strong>Note:</strong> The C# SDK is the official Anthropic SDK for C#. Tool use is supported via the Messages API. 
A class-annotation-based tool runner is not available; use raw tool definitions with JSON schema. 
The SDK also supports Microsoft.Extensions.AI <code>IChatClient</code> integration with function invocation.
</p>

<hr>

<h2>Installation</h2>

<pre>
dotnet add package Anthropic
</pre>

<hr>

<h2>Client Initialization</h2>

<pre>
using Anthropic;

// Default (uses ANTHROPIC_API_KEY env var)
AnthropicClient client = new();

// Explicit API key (use environment variables — never hardcode keys)
AnthropicClient client = new() {
    ApiKey = Environment.GetEnvironmentVariable("ANTHROPIC_API_KEY")
};
</pre>

<hr>

<h2>Basic Message Request</h2>

<pre>
using Anthropic.Models.Messages;

var parameters = new MessageCreateParams
{
    Model = Model.ClaudeOpus4_6,
    MaxTokens = 1024,
    Messages = [new() { Role = Role.User, Content = "What is the capital of France?" }]
};

var message = await client.Messages.Create(parameters);

Console.WriteLine(message);
</pre>

<hr>

<h2>Streaming</h2>

<pre>
using Anthropic.Models.Messages;

var parameters = new MessageCreateParams
{
    Model = Model.ClaudeOpus4_6,
    MaxTokens = 1024,
    Messages = [new() { Role = Role.User, Content = "Write a haiku" }]
};

await foreach (RawMessageStreamEvent streamEvent in client.Messages.CreateStreaming(parameters))
{
    if (streamEvent.TryPickContentBlockDelta(out var delta) &&
        delta.Delta.TryPickText(out var text))
    {
        Console.Write(text.Text);
    }
}
</pre>

<hr>

<h2>Tool Use (Manual Loop)</h2>

<p>
The C# SDK supports raw tool definitions using JSON schema.
</p>

<p>
For the complete tool definition format and agentic loop pattern, see the shared documentation:
</p>

<pre>
../shared/tool-use-concepts.md
</pre>
