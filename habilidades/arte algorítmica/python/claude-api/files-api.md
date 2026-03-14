# Files API — Python

O Files API permite **fazer upload de arquivos** para uso em requisições da Messages API.  
Use `file_id` em blocos de conteúdo para **referenciar arquivos** sem precisar reenvia-los várias vezes.

> **Beta:** passe `betas=["files-api-2025-04-14"]` nas chamadas.

---

## Key Facts

- Tamanho máximo de arquivo: **500 MB**
- Armazenamento total: **100 GB por organização**
- Arquivos permanecem até serem deletados
- Operações com arquivos são gratuitas; apenas tokens em mensagens são cobrados
- Não disponível em Amazon Bedrock ou Google Vertex AI

---

## Upload a File

```python
import anthropic

client = anthropic.Anthropic()

uploaded = client.beta.files.upload(
    file=("report.pdf", open("report.pdf", "rb"), "application/pdf"),
)

print(uploaded.id)
print(uploaded.size_bytes)
```

---

## Use a File in Messages

### PDF / Text Document

```python
response = client.beta.messages.create(
    model="claude-opus-4-6",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": [
            {"type": "text", "text": "Summarize the key findings in this report."},
            {
                "type": "document",
                "source": {"type": "file", "file_id": uploaded.id},
                "title": "Q4 Report",
                "citations": {"enabled": True}
            }
        ]
    }],
    betas=["files-api-2025-04-14"],
)

print(response.content[0].text)
```

### Image

```python
image_file = client.beta.files.upload(
    file=("photo.png", open("photo.png", "rb"), "image/png"),
)

response = client.beta.messages.create(
    model="claude-opus-4-6",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": [
            {"type": "text", "text": "What's in this image?"},
            {
                "type": "image",
                "source": {"type": "file", "file_id": image_file.id}
            }
        ]
    }],
    betas=["files-api-2025-04-14"],
)
```

---

## Manage Files

### List Files

```python
files = client.beta.files.list()

for f in files.data:
    print(f.id, f.filename, f.size_bytes)
```

### Get File Metadata

```python
file_info = client.beta.files.retrieve_metadata(
    "file_011CNha8iCJcU1wXNR6q4V8w"
)

print(file_info.filename)
print(file_info.mime_type)
```

### Delete a File

```python
client.beta.files.delete(
    "file_011CNha8iCJcU1wXNR6q4V8w"
)
```

### Download a File

> Apenas arquivos criados por code execution tools ou skills podem ser baixados.

```python
file_content = client.beta.files.download(
    "file_011CNha8iCJcU1wXNR6q4V8w"
)

file_content.write_to_file("output.txt")
```

---

## Full End-to-End Example

> Faça upload de um documento uma vez e faça múltiplas perguntas.

```python
import anthropic

client = anthropic.Anthropic()

uploaded = client.beta.files.upload(
    file=("contract.pdf", open("contract.pdf", "rb"), "application/pdf"),
)

questions = [
    "What are the key terms and conditions?",
    "What is the termination clause?",
    "Summarize the payment schedule.",
]

for question in questions:

    response = client.beta.messages.create(
        model="claude-opus-4-6",
        max_tokens=1024,
        messages=[{
            "role": "user",
            "content": [
                {"type": "text", "text": question},
                {
                    "type": "document",
                    "source": {"type": "file", "file_id": uploaded.id}
                }
            ]
        }],
        betas=["files-api-2025-04-14"],
    )

    print(question)
    print(response.content[0].text[:200])

client.beta.files.delete(uploaded.id)
```
