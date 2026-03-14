<div align="center">

<h1>Claude API — Java</h1>

<p><b>Official Java SDK for the Claude API</b></p>

</div>

<hr>

<p>
<strong>Note:</strong> The Java SDK supports the Claude API and beta tool use with annotated classes.
The Agent SDK is not yet available for Java.
</p>

<hr>

<h2>Installation</h2>

<h3>Maven</h3>

<pre>
&lt;dependency&gt;
    &lt;groupId&gt;com.anthropic&lt;/groupId&gt;
    &lt;artifactId&gt;anthropic-java&lt;/artifactId&gt;
    &lt;version&gt;2.15.0&lt;/version&gt;
&lt;/dependency&gt;
</pre>

<h3>Gradle</h3>

<pre>
implementation("com.anthropic:anthropic-java:2.15.0")
</pre>

<hr>

<h2>Client Initialization</h2>

<pre>
import com.anthropic.client.AnthropicClient;
import com.anthropic.client.okhttp.AnthropicOkHttpClient;

// Default (reads ANTHROPIC_API_KEY from environment)
AnthropicClient client = AnthropicOkHttpClient.fromEnv();

// Explicit API key
AnthropicClient client = AnthropicOkHttpClient.builder()
    .apiKey("your-api-key")
    .build();
</pre>

<hr>

<h2>Basic Message Request</h2>

<pre>
import com.anthropic.models.messages.MessageCreateParams;
import com.anthropic.models.messages.Message;
import com.anthropic.models.messages.Model;

MessageCreateParams params = MessageCreateParams.builder()
    .model(Model.CLAUDE_OPUS_4_6)
    .maxTokens(1024L)
    .addUserMessage("What is the capital of France?")
    .build();

Message response = client.messages().create(params);

response.content().stream()
    .flatMap(block -> block.text().stream())
    .forEach(textBlock -> System.out.println(textBlock.text()));
</pre>

<hr>

<h2>Streaming</h2>

<pre>
import com.anthropic.core.http.StreamResponse;
import com.anthropic.models.messages.RawMessageStreamEvent;

MessageCreateParams params = MessageCreateParams.builder()
    .model(Model.CLAUDE_OPUS_4_6)
    .maxTokens(1024L)
    .addUserMessage("Write a haiku")
    .build();

try (StreamResponse&lt;RawMessageStreamEvent&gt; streamResponse =
        client.messages().createStreaming(params)) {

    streamResponse.stream()
        .flatMap(event -> event.contentBlockDelta().stream())
        .flatMap(deltaEvent -> deltaEvent.delta().text().stream())
        .forEach(textDelta -> System.out.print(textDelta.text()));
}
</pre>

<hr>

<h2>Tool Use (Beta)</h2>

<p>
The Java SDK supports beta tool use with annotated classes.
Tool classes implement <code>Supplier&lt;String&gt;</code> for automatic execution via <code>BetaToolRunner</code>.
</p>

<h3>Tool Runner (Automatic Loop)</h3>

<pre>
import com.anthropic.models.beta.messages.MessageCreateParams;
import com.anthropic.models.beta.messages.BetaMessage;
import com.anthropic.helpers.BetaToolRunner;
import com.fasterxml.jackson.annotation.JsonClassDescription;
import com.fasterxml.jackson.annotation.JsonPropertyDescription;
import java.util.function.Supplier;

@JsonClassDescription("Get the weather in a given location")
static class GetWeather implements Supplier&lt;String&gt; {

    @JsonPropertyDescription("The city and state, e.g. San Francisco, CA")
    public String location;

    @Override
    public String get() {
        return "The weather in " + location + " is sunny and 72°F";
    }
}

BetaToolRunner toolRunner = client.beta().messages().toolRunner(
    MessageCreateParams.builder()
        .model("claude-opus-4-6")
        .maxTokens(1024L)
        .putAdditionalHeader("anthropic-beta", "structured-outputs-2025-11-13")
        .addTool(GetWeather.class)
        .addUserMessage("What's the weather in San Francisco?")
        .build());

for (BetaMessage message : toolRunner) {
    System.out.println(message);
}
</pre>

<hr>

<h3>Non-Beta Tool Use</h3>

<p>
Tool use is also available through the non-beta
<code>com.anthropic.models.messages.MessageCreateParams</code>
using <code>addTool(Tool)</code> for manually defined JSON schemas.
</p>

<p>
This method does not require the beta namespace.
The beta namespace is only required for the annotation-based
convenience layer (<code>@JsonClassDescription</code>, <code>BetaToolRunner</code>).
</p>

<hr>

<h3>Manual Loop</h3>

<p>
For manual tool loops:
</p>

<ul>

<li>Define tools as JSON schema in the request</li>

<li>Handle <code>tool_use</code> blocks in the response</li>

<li>Send <code>tool_result</code> back to Claude</li>

<li>Repeat until <code>stop_reason = "end_turn"</code></li>

</ul>

<p>
See the shared documentation for the agentic loop pattern:
</p>

<pre>
../shared/tool-use-concepts.md
</pre>
