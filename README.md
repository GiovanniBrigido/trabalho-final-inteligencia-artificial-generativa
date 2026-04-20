<div align="center">
  <h1>🤖 Consultor Inteligente de Sistemas</h1>
  <p><em>Pipeline RAG Multi-Agente para Transformação Digital na SECOM</em></p>
</div>

---

## 📑 Índice

- [Sobre o Projeto](#-sobre-o-projeto)
- [Pré-requisitos](#-pré-requisitos)
- [Estrutura do Repositório](#-estrutura-do-repositório)
- [Como Usar](#-como-usar)
  - [1. Siga os passos do professor](#1-siga-os-passos-do-professor)
  - [2. Importe os pipelines no Dify](#2-importe-os-pipelines-no-dify)
  - [3. Configure a base de conhecimento](#3-configure-a-base-de-conhecimento)
  - [4. Teste os agentes](#4-teste-os-agentes)
- [Agentes e Pipelines](#-agentes-e-pipelines)
- [Modelos Utilizados](#-modelos-utilizados)
- [Ponto Extra — Consultor de Melhoria](#-ponto-extra--consultor-de-melhoria)

---

## 📖 Sobre o Projeto

Na SECOM/DSIC, servidores dependem de colegas experientes para esclarecer dúvidas operacionais sobre o sistema **MidiaCad** — plataforma de gestão de contratos de publicidade. A documentação técnica existe mas não está acessível de forma conversacional, gerando dependência de consultas presenciais e curva de aprendizado lenta.

Este repositório apresenta o **Consultor MidiaCad**: pipeline RAG multi-agente 100% local, desenvolvido para responder dúvidas operacionais com base em documentação técnica indexada. O projeto inclui ainda dois sistemas complementares:

- **Verificador de TR** — analisa Termos de Referência com base na Lei 14.133/2021, com envio automatizado de relatórios via Gmail (MCP).
- **Consultor PL 2338/2023** — demonstra empiricamente a superioridade de RAG sobre LLM puro em domínio jurídico específico.

> 📄 A apresentação completa do projeto está disponível na pasta [`/agentes/pptx/`](./agentes/pptx/).

---

## 🛠 Pré-requisitos

As seguintes ferramentas são necessárias:

1. **Docker** e **Docker Compose**
2. **Dify** (implantado via Docker — veja as instruções do professor)
3. **Ollama** com os modelos:
   - `llama3.2:3b` (LLM local para o Consultor MidiaCad)
   - `qwen3-embedding:4b` (embedding local)
4. **Gmail MCP Server** (necessário apenas para o Verificador de TR)
5. **OpenRouter** com acesso ao `Gemini 2.5 Flash Lite` (para Verificador de TR e Consultor PL) ou outra API de LLM válida, como ChatGPT ou Antrophic


---

## 🚀 Como Usar

### 1. Siga os passos do professor

Configure toda a infraestrutura base (Docker, Dify, Gmail MCP Server, modelos de embedding e reranker) seguindo o repositório do professor:

📂 **[`/agentes/`](./agentes/README.md)**

> **Importante:** após configurar a infraestrutura conforme as instruções do professor, volte aqui para importar os pipelines deste projeto.

---

### 2. Importe os pipelines no Dify

Com o Dify em execução, importe todos os pipelines disponíveis na pasta [`/agentes/pipelines-dify/`](./agentes/pipelines-dify/):

| Pipeline | Arquivo |
|---|---|
| Consultor MidiaCad | `MidiaCad - Casos de Uso.yml` |
| Consultor PL 2338/2023 | `Consultar PL - I.A..yml` |
| Verificador de TR | `Verificador de TR.yml` |
| Consultor de Melhoria *(ponto extra)* | `Consultor de Melhoria.yml` |

Para importar: no Dify, acesse **Studio** → **Import from DSL file** e selecione cada `.yml`.

---

### 3. Configure a base de conhecimento

Siga as instruções de criação de base de conhecimento conforme o repositório do professor. Ao criar o Knowledge, faça upload do seguinte arquivo:

📄 **[`/agentes/knowledge/PL_IA.pdf`](./agentes/knowledge/PL_IA.pdf)** — texto completo do PL 2338/2023

> Para o **Consultor MidiaCad**, a base de conhecimento é composta pelos documentos `.docx` dos casos de uso do MidiaCad (pasta [`/agentes/casos-de-uso/`](./agentes/casos-de-uso/)). Siga o mesmo processo de upload e indexação.

**Parâmetros de chunking recomendados:**

| Parâmetro | Valor |
|---|---|
| Estratégia | Por parágrafo (`\n\n`) |
| Tamanho máximo do chunk | 3.200 caracteres |
| Overlap | 500 caracteres |
| Index Method | High Quality |
| Top K | 4 |
| Score Threshold | 0,4 |

---

### 4. Teste os agentes

Após a importação e configuração, teste cada pipeline com as perguntas de exemplo disponíveis em:

📄 **[`/agentes/exemplos/Perguntas_Teste_Agentes.md`](./agentes/exemplos/Perguntas_Teste_Agentes.md)**

> ⚠️ **Atenção:** o pipeline **MidiaCad - Casos de Uso** não pode ser testado externamente pois acessa documentação interna sensível da SECOM/DSIC. Os demais pipelines podem ser testados livremente.

---

## 🤖 Agentes e Pipelines

### 🔵 Consultor MidiaCad — `MidiaCad - Casos de Uso.yml`

Pipeline RAG multi-agente 100% local para responder dúvidas operacionais sobre o sistema MidiaCad.

```
Entrada (pergunta) → Recuperação de Contexto (RAG) → Interpretador (LLM)
→ Limpador de Siglas (Python) → Construtor (LLM) → Resposta
```

- **Dois LLMs especializados:** Interpretador de intenção + Construtor de resposta
- **Nó Python:** normalização determinística de siglas técnicas (DSIC, UC, ECU…)
- **Guard-rail de incerteza:** redireciona o usuário a um especialista quando a documentação é insuficiente
- **Processamento 100% local:** Llama 3.2:3b via Ollama, sem APIs externas

---

### 🟢 Consultor PL 2338/2023 — `Consultar PL - I.A..yml`

Prova de conceito comparativa que exibe lado a lado a resposta de **LLM puro** vs. **LLM com RAG** para a mesma pergunta sobre o PL 2338/2023.

---

### 🔴 Verificador de TR — `Verificador de TR.yml`

Analisa Termos de Referência de licitações públicas com base na Lei 14.133/2021. Exige o Gmail MCP Server em execução para o envio automatizado de relatórios por e-mail.

```
Entrada TR → Extrator de Texto → Recuperação Legal (RAG)
→ Analista Técnico → Auditor de Risco → Consultor de Melhoria
→ Gerador de Contexto (Python) → Agente Carteiro (Gmail MCP) → Saída
```

---

### 🟡 Consultor de Melhoria — `Consultor de Melhoria.yml` *(Ponto Extra)*

Agente que recebe o TR e os relatórios do Analista Técnico e do Auditor de Risco, e gera um relatório com sugestões de melhorias e redações alternativas para os problemas identificados. Veja evidências na pasta [`/ponto-extra_consultor-melhoria/`](./ponto-extra_consultor-melhoria/).

---

## ⚙️ Modelos Utilizados

Os modelos abaixo foram utilizados conforme descrito no artigo científico do projeto.

| Componente | Modelo | Onde |
|---|---|---|
| LLM — Consultor MidiaCad | `llama3.2:3b` | Ollama (local) |
| Embedding | `qwen3-embedding:4b` | Ollama (local) |
| Reranker | `BAAI/bge-reranker-v2-m3` | Docker TEI |
| LLM — Verificador de TR e Consultor PL | `Gemini 2.5 Flash Lite` | OpenRouter |

---

## 🏆 Ponto Extra — Consultor de Melhoria

O agente **Consultor de Melhoria** foi desenvolvido como exercício extra da disciplina. Ele recebe o TR original e os relatórios gerados pelo Analista Técnico e pelo Auditor de Risco, e produz um relatório de melhorias com sugestões e redações alternativas para os problemas identificados.

O pipeline está disponível em [`/agentes/pipelines-dify/Consultor de Melhoria.yml`](./agentes/pipelines-dify/) e as evidências de execução estão em [`/ponto-extra_consultor-melhoria/`](./ponto-extra_consultor-melhoria/).

---

<div align="center">
  <p>
    <strong>Giovanni Brigido Bezerra Cardoso | Paulo Pita</strong><br>
    Instituto Brasileiro de Ensino, Desenvolvimento e Pesquisa (IDP)<br>
    Disciplina de Inteligência Generativa — Brasília, DF — Abril de 2026
  </p>
</div>