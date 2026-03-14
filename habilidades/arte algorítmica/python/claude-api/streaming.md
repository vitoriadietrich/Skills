# Streaming — Python

O Streaming API permite **receber respostas incrementalmente** da Messages API, ideal para visualização em tempo real ou respostas longas.

---

## Quick Start

```python
with client.messages.stream(
    model="claude-opus-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Write a story"}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

### Async

```python
async with async_client.messages.stream(
    model="claude-opus-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Write a story"}]
) as stream:
    async for text in stream.text_stream:
        print(text, end="", flush=True)
```

---

## Handling Different Content Types

Claude pode retornar **textos, blocos de thinking ou tool use**. Trate cada um apropriadamente:

> **Opus 4.6:** use `thinking: {type: "adaptive"}`.  
> Em modelos antigos, use `thinking: {type: "enabled", budget_tokens: N}`.

```python
with client.messages.stream(
    model="claude-opus-4-6",
    max_tokens=16000,
    thinking={"type": "adaptive"},
    messages=[{"role": "user", "content": "Analyze this problem"}]
) as stream:
    for event in stream:
        if event.type == "content_block_start":
            if event.content_block.type == "thinking":
                print("\n[Thinking...]")
            elif event.content_block.type == "text":
                print("\n[Response:]")

        elif event.type == "content_block_delta":
            if event.delta.type == "thinking_delta":
                print(event.delta.thinking, end="", flush=True)
            elif event.delta.type == "text_delta":
                print(event.delta.text, end="", flush=True)
```

---

## Streaming with Tool Use

Para **usar ferramentas**, o Python tool runner retorna mensagens completas. Use streaming em um loop manual para streaming token a token:

```python
with client.messages.stream(
    model="claude-opus-4-6",
    max_tokens=4096,
    tools=tools,
    messages=messages
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)

    response = stream.get_final_message()
    # Continue se response.stop_reason == "tool_use"
```

---

## Getting the Final Message

```python
with client.messages.stream(
    model="claude-opus-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello"}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)

    final_message = stream.get_final_message()
    print(f"\n\nTokens used: {final_message.usage.output_tokens}")
```

---

## Streaming with Progress Updates

```python
def stream_with_progress(client, **kwargs):
    """Stream a response with progress updates."""
    total_tokens = 0
    content_parts = []

    with client.messages.stream(**kwargs) as stream:
        for event in stream:
            if event.type == "content_block_delta":
                if event.delta.type == "text_delta":
                    text = event.delta.text
                    content_parts.append(text)
                    print(text, end="", flush=True)

            elif event.type == "message_delta":
                if event.usage and event.usage.output_tokens is not None:
                    total_tokens = event.usage.output_tokens

        final_message = stream.get_final_message()

    print(f"\n\n[Tokens used: {total_tokens}]")
    return "".join(content_parts)
```

---

## Error Handling in Streams

```python
try:
    with client.messages.stream(
        model="claude-opus-4-6",
        max_tokens=1024,
        messages=[{"role": "user", "content": "Write a story"}]
    ) as stream:
        for text in stream.text_stream:
            print(text, end="", flush=True)
except anthropic.APIConnectionError:
    print("\nConnection lost. Please retry.")
except anthropic.RateLimitError:
    print("\nRate limited. Please wait and retry.")
except anthropic.APIStatusError as e:
    print(f"\nAPI error: {e.status_code}")
```

---

## Stream Event Types

| Event Type           | Description                      | When it fires                        |
|---------------------|----------------------------------|-------------------------------------|
| message_start        | Contém metadata da mensagem      | Uma vez no início                   |
| content_block_start  | Novo bloco de conteúdo           | Ao iniciar um bloco de texto/tool   |
| content_block_delta  | Atualização incremental          | Para cada token ou chunk            |
| content_block_stop   | Bloco de conteúdo completo       | Ao terminar o bloco                 |
| message_delta        | Atualizações em nível de mensagem| Contém stop_reason e usage          |
| message_stop         | Mensagem completa                | Uma vez no final                     |

---

## Best Practices

1. **Always flush output** — use `flush=True` para mostrar tokens imediatamente
2. **Handle partial responses** — se o stream for interrompido, o conteúdo pode ficar incompleto
3. **Track token usage** — o evento `message_delta` contém informações de uso
4. **Use timeouts** — defina timeouts adequados para sua aplicação
5. **Default to streaming** — use `.get_final_message()` para obter a resposta completa, mesmo durante streaming, protegendo contra timeouts
