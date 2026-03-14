<div align="center">

<h1>Habilidades</h1>

</div>

<p>
As habilidades são pastas de instruções, scripts e recursos que o Claude carrega dinamicamente para melhorar o desempenho em tarefas especializadas. As habilidades ensinam o Claude a concluir tarefas específicas de forma repetível, seja criando documentos com as diretrizes de marca da sua empresa, analisando dados usando os fluxos de trabalho específicos da sua organização ou automatizando tarefas pessoais.
</p>

<p>Para obter mais informações, consulte:</p>

<ul>
<li>O que são habilidades?</li>
<li>Utilizando habilidades em Claude</li>
<li>Como criar habilidades personalizadas</li>
<li>Capacitando agentes para o mundo real com Habilidades de Agente.</li>
</ul>

<hr>

<h1>Sobre este repositório</h1>

<p>
Este repositório contém habilidades que demonstram o que é possível fazer com o sistema de habilidades do Claude. Essas habilidades variam de aplicações criativas (arte, música, design) a tarefas técnicas (testes de aplicativos web, geração de servidores MCP) e fluxos de trabalho corporativos (comunicação, branding, etc.).
</p>

<p>
Cada habilidade está contida em sua própria pasta, com um <code>SKILL.md</code> arquivo contendo as instruções e os metadados que Claude utiliza. Navegue por essas habilidades para se inspirar e criar as suas próprias ou para entender diferentes padrões e abordagens.
</p>

<p>
Muitas das habilidades neste repositório são de código aberto (Apache 2.0). Também incluímos as habilidades de criação e edição de documentos que alimentam os recursos de documentos do Claude nos bastidores, nas subpastas:
</p>

<pre>
skills/docx
skills/pdf
skills/pptx
skills/xlsx
</pre>

<p>
Essas habilidades têm código-fonte disponível, não são de código aberto, mas queríamos compartilhá-las com os desenvolvedores como referência para habilidades mais complexas que são ativamente usadas em uma aplicação de IA em produção.
</p>

<hr>

<h1>Isenção de responsabilidade</h1>

<p>
Essas habilidades são fornecidas apenas para fins de demonstração e educacionais. Embora algumas dessas funcionalidades possam estar disponíveis no Claude, as implementações e os comportamentos que você recebe do Claude podem diferir do que é mostrado nessas habilidades. Essas habilidades servem para ilustrar padrões e possibilidades. Sempre teste as habilidades minuciosamente em seu próprio ambiente antes de utilizá-las para tarefas críticas.
</p>

<hr>

<h1>Conjunto de habilidades</h1>

<pre>
./skills
</pre>

<p>
Exemplos de habilidades para Criação e Design, Desenvolvimento e Técnica, Empresarial e Comunicação, e Documentação.
</p>

<pre>
./spec
</pre>

<p>
Especificação de Habilidades do Agente
</p>

<pre>
./template
</pre>

<p>
Modelo de habilidade
</p>

<hr>

<h1>Experimente no Claude Code, Claude.ai e na API</h1>

<h2>Código Claude</h2>

<p>
Você pode registrar este repositório como um marketplace de plugins do Claude Code executando o seguinte comando no Claude Code:
</p>

<pre><code>
/plugin marketplace add anthropics/skills
</code></pre>

<p>Em seguida, para instalar um conjunto específico de habilidades:</p>

<ol>
<li>Selecione Browse and install plugins</li>
<li>Selecione anthropic-agent-skills</li>
<li>Selecione document-skills ou example-skills</li>
<li>Selecione Install now</li>
</ol>

<p>Alternativamente, instale qualquer um dos plugins diretamente através de:</p>

<pre><code>
/plugin install document-skills@anthropic-agent-skills
/plugin install example-skills@anthropic-agent-skills
</code></pre>

<p>
Após instalar o plugin, você pode usar a habilidade simplesmente mencionando-a. Por exemplo, se você instalar o plugin document-skills pelo marketplace, pode pedir ao Claude Code para fazer algo como:
</p>

<pre><code>
Use a habilidade PDF para extrair os campos do formulário de path/to/some-file.pdf
</code></pre>

<hr>

<h1>Claude.ai</h1>

<p>
Todas essas habilidades de exemplo já estão disponíveis nos planos pagos do Claude.ai.
</p>

<p>
Para usar qualquer habilidade deste repositório ou carregar habilidades personalizadas, siga as instruções em Usando habilidades no Claude.
</p>

<hr>

<h1>API Claude</h1>

<p>
Você pode usar as habilidades pré-construídas da Anthropic e fazer upload de habilidades personalizadas por meio da API do Claude. Consulte o Guia de Início Rápido da API de Habilidades para obter mais informações.
</p>

<hr>

<h1>Criando uma habilidade básica</h1>

<p>
Criar skills é simples: basta uma pasta com um SKILL.md arquivo contendo o código YAML e as instruções. Você pode usar a skill modelo deste repositório como ponto de partida:
</p>

<pre><code>
---
name: my-skill-name
description: A clear description of what this skill does and when to use it
---

# My Skill Name

[Add your instructions here that Claude will follow when this skill is active]

## Examples
- Example usage 1
- Example usage 2

## Guidelines
- Guideline 1
- Guideline 2
</code></pre>

<p>O frontmatter requer apenas dois campos:</p>

<ul>
<li><b>name</b> - Um identificador único para sua habilidade (letras minúsculas, hífens para espaços)</li>
<li><b>description</b> - Uma descrição completa da função da habilidade e quando usá-la.</li>
</ul>

<p>
O conteúdo em Markdown abaixo contém as instruções, exemplos e diretrizes que Claude seguirá. Para mais detalhes, consulte Como criar habilidades personalizadas.
</p>

<hr>

<h1>Habilidades de parceria</h1>

<p>
As habilidades são uma ótima maneira de ensinar Claude a usar melhor softwares específicos. Ao observarmos exemplos incríveis de habilidades demonstradas por nossos parceiros, podemos destacar algumas delas aqui:
</p>

<ul>
<li>Notion - Habilidades do Notion para Claude</li>
</ul>
