<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Claude API — Ruby</title>

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

<h1>Claude API — Ruby</h1>

<blockquote>
<strong>Note:</strong> The Ruby SDK supports the Claude API. A tool runner is available in beta via <code>client.beta.messages.tool_runner()</code>. Agent SDK is not yet available for Ruby.
</blockquote>

<h2>Installation</h2>

<pre><code>gem install anthropic</code></pre>

<h2>Client Initialization</h2>

<pre><code>require "anthropic"

# Default (uses ANTHROPIC_API_KEY env var)
client = Anthropic::Client.new

# Explicit API key
client = Anthropic::Client.new(api_key: "your-api-key")
</code></pre>

<hr>

<h2>Basic Message Request</h2>

<pre><code>message = client.messages.create(
  model: :"claude-opus-4-6",
  max_tokens: 1024,
  messages: [
    { role: "user", content: "What is the capital of France?" }
  ]
)
puts message.content.first.text
</code></pre>

<hr>

<h2>Streaming</h2>

<pre><code>stream = client.messages.stream(
  model: :"claude-opus-4-6",
  max_tokens: 1024,
  messages: [{ role: "user", content: "Write a haiku" }]
)

stream.text.each { |text| print(text) }
</code></pre>

<hr>

<h2>Tool Use</h2>

<p>The Ruby SDK supports tool use via raw JSON schema definitions and also provides a beta tool runner for automatic tool execution.</p>

<h3>Tool Runner (Beta)</h3>

<pre><code>class GetWeatherInput &lt; Anthropic::BaseModel
  required :location, String, doc: "City and state, e.g. San Francisco, CA"
end

class GetWeather &lt; Anthropic::BaseTool
  doc "Get the current weather for a location"

  input_schema GetWeatherInput

  def call(input)
    "The weather in #{input.location} is sunny and 72°F."
  end
end

client.beta.messages.tool_runner(
  model: :"claude-opus-4-6",
  max_tokens: 1024,
  tools: [GetWeather.new],
  messages: [{ role: "user", content: "What's the weather in San Francisco?" }]
).each_message do |message|
  puts message.content
end
</code></pre>

<h3>Manual Loop</h3>

<p>See the shared tool use concepts for the tool definition format and agentic loop pattern.</p>

</body>
</html>
