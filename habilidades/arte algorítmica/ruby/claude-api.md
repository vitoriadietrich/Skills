# Claude API — Ruby

> **Note:** O SDK Ruby suporta a Claude API.  
> Um tool runner está disponível em beta via `client.beta.messages.tool_runner()`.  
> O Agent SDK ainda não está disponível para Ruby.

---

## Installation

```bash
gem install anthropic
```

---

## Client Initialization

```ruby
require "anthropic"

# Default (usa a variável de ambiente ANTHROPIC_API_KEY)
client = Anthropic::Client.new

# Chave de API explícita
client = Anthropic::Client.new(api_key: "your-api-key")
```

---

## Basic Message Request

```ruby
message = client.messages.create(
  model: :"claude-opus-4-6",
  max_tokens: 1024,
  messages: [
    { role: "user", content: "What is the capital of France?" }
  ]
)

puts message.content.first.text
```

---

## Streaming

```ruby
stream = client.messages.stream(
  model: :"claude-opus-4-6",
  max_tokens: 1024,
  messages: [{ role: "user", content: "Write a haiku" }]
)

stream.text.each { |text| print(text) }
```

---

## Tool Use

O SDK Ruby suporta uso de ferramentas via definições de JSON schema e também fornece um tool runner beta para execução automática.

### Tool Runner (Beta)

```ruby
class GetWeatherInput < Anthropic::BaseModel
  required :location, String, doc: "City and state, e.g. San Francisco, CA"
end

class GetWeather < Anthropic::BaseTool
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
```

### Manual Loop

Veja o documento `shared/tool-use-concepts.md` para o formato de definição de ferramentas e padrão do agentic loop.
