<div align="center">

<h1>Claude API — PHP</h1>

<p><b>Official PHP SDK for the Claude API</b></p>

</div>

<hr>

<p>
<strong>Note:</strong> The PHP SDK is the official Anthropic SDK for PHP. 
Tool runner and Agent SDK are not available. 
Bedrock, Vertex AI, and Foundry clients are supported.
</p>

<hr>

<h2>Installation</h2>

<pre>
composer require "anthropic-ai/sdk"
</pre>

<hr>

<h2>Client Initialization</h2>

<pre>
use Anthropic\Client;

// Using API key from environment variable
$client = new Client(apiKey: getenv("ANTHROPIC_API_KEY"));
</pre>

<hr>

<h3>Amazon Bedrock</h3>

<pre>
use Anthropic\BedrockClient;

$client = new BedrockClient(
    region: 'us-east-1',
);
</pre>

<hr>

<h3>Google Vertex AI</h3>

<pre>
use Anthropic\VertexClient;

$client = new VertexClient(
    region: 'us-east5',
    projectId: 'my-project-id',
);
</pre>

<hr>

<h3>Anthropic Foundry</h3>

<pre>
use Anthropic\FoundryClient;

$client = new FoundryClient(
    authToken: getenv("ANTHROPIC_AUTH_TOKEN"),
);
</pre>

<hr>

<h2>Basic Message Request</h2>

<pre>
$message = $client->messages->create(
    model: 'claude-opus-4-6',
    maxTokens: 1024,
    messages: [
        ['role' => 'user', 'content' => 'What is the capital of France?'],
    ],
);

echo $message->content[0]->text;
</pre>

<hr>

<h2>Streaming</h2>

<pre>
$stream = $client->messages->createStream(
    model: 'claude-opus-4-6',
    maxTokens: 1024,
    messages: [
        ['role' => 'user', 'content' => 'Write a haiku'],
    ],
);

foreach ($stream as $event) {
    echo $event;
}
</pre>

<hr>

<h2>Tool Use (Manual Loop)</h2>

<p>
The PHP SDK supports raw tool definitions using JSON schema.
</p>

<p>
For the tool definition format and agentic loop pattern, see:
</p>

<pre>
../shared/tool-use-concepts.md
</pre>
