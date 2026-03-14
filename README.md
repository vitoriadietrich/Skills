Habilidades
As habilidades são pastas de instruções, scripts e recursos que o Claude carrega dinamicamente para melhorar o desempenho em tarefas especializadas. As habilidades ensinam o Claude a concluir tarefas específicas de forma repetível, seja criando documentos com as diretrizes de marca da sua empresa, analisando dados usando os fluxos de trabalho específicos da sua organização ou automatizando tarefas pessoais.

Para obter mais informações, consulte:

O que são habilidades?
Utilizando habilidades em Claude
Como criar habilidades personalizadas
Capacitando agentes para o mundo real com Habilidades de Agente.
Sobre este repositório
Este repositório contém habilidades que demonstram o que é possível fazer com o sistema de habilidades do Claude. Essas habilidades variam de aplicações criativas (arte, música, design) a tarefas técnicas (testes de aplicativos web, geração de servidores MCP) e fluxos de trabalho corporativos (comunicação, branding, etc.).

Cada habilidade está contida em sua própria pasta, com um SKILL.mdarquivo contendo as instruções e os metadados que Claude utiliza. Navegue por essas habilidades para se inspirar e criar as suas próprias ou para entender diferentes padrões e abordagens.

Muitas das habilidades neste repositório são de código aberto (Apache 2.0). Também incluímos as habilidades de criação e edição de documentos que alimentam os recursos de documentos do Claude nos bastidores, nas subpastas skills/docx` <nome_da_pasta> skills/pdf`, ` <nome_da_pasta>`, `<nome_da_pasta> skills/pptx` e `<nome_da_pasta>` skills/xlsx. Essas habilidades têm código-fonte disponível, não são de código aberto, mas queríamos compartilhá-las com os desenvolvedores como referência para habilidades mais complexas que são ativamente usadas em uma aplicação de IA em produção.

Isenção de responsabilidade
Essas habilidades são fornecidas apenas para fins de demonstração e educacionais. Embora algumas dessas funcionalidades possam estar disponíveis no Claude, as implementações e os comportamentos que você recebe do Claude podem diferir do que é mostrado nessas habilidades. Essas habilidades servem para ilustrar padrões e possibilidades. Sempre teste as habilidades minuciosamente em seu próprio ambiente antes de utilizá-las para tarefas críticas.

Conjunto de habilidades
./skills : Exemplos de habilidades para Criação e Design, Desenvolvimento e Técnica, Empresarial e Comunicação, e Documentação.
./spec : Especificação de Habilidades do Agente
./template : Modelo de habilidade
Experimente no Claude Code, Claude.ai e na API.
Código Claude
Você pode registrar este repositório como um marketplace de plugins do Claude Code executando o seguinte comando no Claude Code:

/plugin marketplace add anthropics/skills
Em seguida, para instalar um conjunto específico de habilidades:

SelecioneBrowse and install plugins
Selecioneanthropic-agent-skills
Selecione document-skillsouexample-skills
SelecioneInstall now
Alternativamente, instale qualquer um dos plugins diretamente através de:

/plugin install document-skills@anthropic-agent-skills
/plugin install example-skills@anthropic-agent-skills
Após instalar o plugin, você pode usar a habilidade simplesmente mencionando-a. Por exemplo, se você instalar o document-skillsplugin pelo marketplace, pode pedir ao Claude Code para fazer algo como: "Use a habilidade PDF para extrair os campos do formulário de path/to/some-file.pdf"

Claude.ai
Todas essas habilidades de exemplo já estão disponíveis nos planos pagos do Claude.ai.

Para usar qualquer habilidade deste repositório ou carregar habilidades personalizadas, siga as instruções em Usando habilidades no Claude .

API Claude
Você pode usar as habilidades pré-construídas da Anthropic e fazer upload de habilidades personalizadas por meio da API do Claude. Consulte o Guia de Início Rápido da API de Habilidades para obter mais informações.

Criando uma habilidade básica
Criar skills é simples: basta uma pasta com um SKILL.mdarquivo contendo o código YAML e as instruções. Você pode usar a skill modelo deste repositório como ponto de partida:

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
O frontmatter requer apenas dois campos:

name- Um identificador único para sua habilidade (letras minúsculas, hífens para espaços)
description- Uma descrição completa da função da habilidade e quando usá-la.
O conteúdo em Markdown abaixo contém as instruções, exemplos e diretrizes que Claude seguirá. Para mais detalhes, consulte Como criar habilidades personalizadas .

Habilidades de parceria
As habilidades são uma ótima maneira de ensinar Claude a usar melhor softwares específicos. Ao observarmos exemplos incríveis de habilidades demonstradas por nossos parceiros, podemos destacar algumas delas aqui:

Notion - Habilidades do Notion para Claude
