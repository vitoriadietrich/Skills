<!DOCTYPE html>
<html lang="en">

<head>
<meta charset="UTF-8">
<title>Files API — Python</title>

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

ul{
    margin-top:10px;
}

li{
    margin-bottom:6px;
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

<h1>Files API — Python</h1>

<p>
The Files API uploads files for use in Messages API requests. Reference files using 
<b>file_id</b> in content blocks to avoid re-uploading files across multiple API calls.
</p>

<p>
<b>Beta:</b> Pass <code>betas=["files-api-2025-04-14"]</code> in API calls.
</p>

<hr>

<h2>Key Facts</h2>

<ul>
<li>Maximum file size: <b>500 MB</b></li>
<li>Total storage: <b>100 GB per organization</b></li>
<li>Files persist until deleted</li>
<li>File operations are free; tokens used in messages are billed</li>
<li>Not available on Amazon Bedrock or Google Vertex AI</li>
</ul>

<hr>

<h2>Upload a File</h2>

<pre><code>
import anthropic

client = anthropic.Anthropic()

uploaded = client.beta.files.upload(
    file=("report.pdf", open("report.pdf", "rb"), "application/pdf"),
)

print(uploaded.id)
print(uploaded.size_bytes)
</code></pre>

<hr>

<h2>Use a File in Messages</h2>

<h3>PDF / Text Document</h3>

<pre><code>
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
</code></pre>

<h3>Image</h3>

<pre><code>
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
</code></pre>

<hr>

<h2>Manage Files</h2>

<h3>List Files</h3>

<pre><code>
files = client.beta.files.list()

for f in files.data:
    print(f.id, f.filename, f.size_bytes)
</code></pre>

<h3>Get File Metadata</h3>

<pre><code>
file_info = client.beta.files.retrieve_metadata(
    "file_011CNha8iCJcU1wXNR6q4V8w"
)

print(file_info.filename)
print(file_info.mime_type)
</code></pre>

<h3>Delete a File</h3>

<pre><code>
client.beta.files.delete(
    "file_011CNha8iCJcU1wXNR6q4V8w"
)
</code></pre>

<h3>Download a File</h3>

<p>
Only files created by code execution tools or skills can be downloaded.
</p>

<pre><code>
file_content = client.beta.files.download(
    "file_011CNha8iCJcU1wXNR6q4V8w"
)

file_content.write_to_file("output.txt")
</code></pre>

<hr>

<h2>Full End-to-End Example</h2>

<p>Upload a document once and ask multiple questions.</p>

<pre><code>
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
</code></pre>

</body>
</html>
