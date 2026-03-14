<div align="center">

<h1>Claude API — Go</h1>

<p><b>Official Go SDK for the Claude API</b></p>

</div>

<hr>

<p>
<strong>Note:</strong> The Go SDK supports the Claude API and beta tool use with <code>BetaToolRunner</code>.
The Agent SDK is not yet available for Go.
</p>

<hr>

<h2>Installation</h2>

<pre>
go get github.com/anthropics/anthropic-sdk-go
</pre>

<hr>

<h2>Client Initialization</h2>

<pre>
import (
    "github.com/anthropics/anthropic-sdk-go"
    "github.com/anthropics/anthropic-sdk-go/option"
)

// Default (uses ANTHROPIC_API_KEY env var)
client := anthropic.NewClient()

// Explicit API key
client := anthropic.NewClient(
    option.WithAPIKey("your-api-key"),
)
</pre>

<hr>

<h2>Basic Message Request</h2>

<pre>
response, err := client.Messages.New(context.TODO(), anthropic.MessageNewParams{
    Model:     anthropic.ModelClaudeOpus4_6,
    MaxTokens: 1024,
    Messages: []anthropic.MessageParam{
        anthropic.NewUserMessage(anthropic.NewTextBlock("What is the capital of France?")),
    },
})
if err != nil {
    log.Fatal(err)
}

fmt.Println(response.Content[0].Text)
</pre>

<hr>

<h2>Streaming</h2>

<pre>
stream := client.Messages.NewStreaming(context.TODO(), anthropic.MessageNewParams{
    Model:     anthropic.ModelClaudeOpus4_6,
    MaxTokens: 1024,
    Messages: []anthropic.MessageParam{
        anthropic.NewUserMessage(anthropic.NewTextBlock("Write a haiku")),
    },
})

for stream.Next() {
    event := stream.Current()
    switch eventVariant := event.AsAny().(type) {
    case anthropic.ContentBlockDeltaEvent:
        switch deltaVariant := eventVariant.Delta.AsAny().(type) {
        case anthropic.TextDelta:
            fmt.Print(deltaVariant.Text)
        }
    }
}

if err := stream.Err(); err != nil {
    log.Fatal(err)
}
</pre>

<hr>

<h2>Tool Use</h2>

<h3>Tool Runner (Beta — Recommended)</h3>

<p>
<strong>Beta:</strong> The Go SDK provides <code>BetaToolRunner</code> for automatic tool use loops
via the <code>toolrunner</code> package.
</p>

<pre>
import (
    "context"
    "fmt"
    "log"

    "github.com/anthropics/anthropic-sdk-go"
    "github.com/anthropics/anthropic-sdk-go/toolrunner"
)

// Define tool input with jsonschema tags for automatic schema generation
type GetWeatherInput struct {
    City string `json:"city" jsonschema:"required,description=The city name"`
}

// Create a tool with automatic schema generation from struct tags
weatherTool, err := toolrunner.NewBetaToolFromJSONSchema(
    "get_weather",
    "Get current weather for a city",
    func(ctx context.Context, input GetWeatherInput) (anthropic.BetaToolResultBlockParamContentUnion, error) {
        return anthropic.BetaToolResultBlockParamContentUnion{
            OfText: &anthropic.BetaTextBlockParam{
                Text: fmt.Sprintf("The weather in %s is sunny, 72°F", input.City),
            },
        }, nil
    },
)

if err != nil {
    log.Fatal(err)
}

// Create a tool runner that handles the conversation loop automatically
runner := client.Beta.Messages.NewToolRunner(
    []anthropic.BetaTool{weatherTool},
    anthropic.BetaToolRunnerParams{
        BetaMessageNewParams: anthropic.BetaMessageNewParams{
            Model:     anthropic.ModelClaudeOpus4_6,
            MaxTokens: 1024,
            Messages: []anthropic.BetaMessageParam{
                anthropic.NewBetaUserMessage(
                    anthropic.NewBetaTextBlock("What's the weather in Paris?")
                ),
            },
        },
        MaxIterations: 5,
    },
)

// Run until Claude produces a final response
message, err := runner.RunToCompletion(context.Background())

if err != nil {
    log.Fatal(err)
}

fmt.Println(message.Content[0].Text)
</pre>

<hr>

<h3>Key Features of the Go Tool Runner</h3>

<ul>

<li>Automatic schema generation from Go structs via <code>jsonschema</code> tags</li>

<li><code>RunToCompletion()</code> for simple one-shot usage</li>

<li><code>All()</code> iterator for processing each message in the conversation</li>

<li><code>NextMessage()</code> for step-by-step iteration</li>

<li>Streaming variant via <code>NewToolRunnerStreaming()</code> with <code>AllStreaming()</code></li>

</ul>

<hr>

<h3>Manual Loop</h3>

<p>
For fine-grained control, use raw tool definitions via JSON schema.
</p>

<p>
See the shared documentation for the tool definition format and agentic loop pattern:
</p>

<pre>
../shared/tool-use-concepts.md
</pre>
