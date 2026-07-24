# 📚 RAG - Consulta a Documentos Públicos

> Sistema de *Retrieval-Augmented Generation* (RAG) para consulta em linguagem natural a atos normativos, resoluções e editais públicos, com garantia de rastreabilidade e citação das fontes originais.

![Status](https://img.shields.io/badge/Status-Em%20Desenvolvimento-yellow)
![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![FastAPI](https://img.shields.io/badge/FastAPI-0.100%2B-green)

---

## 🎯 Objetivo

Permitir que usuários façam perguntas em português sobre uma base de documentos públicos (ex: resoluções do MPRN, normas internas, editais) e recebam **respostas fundamentadas**. 

O sistema minimiza alucinações de LLMs ao garantir que toda resposta gerada seja estritamente baseada nos trechos recuperados da base vetorial, informando explicitamente **de qual documento e trecho a informação foi extraída**.

---

## 🏗️ Arquitetura do Sistema

O fluxo da aplicação é dividido em duas etapas principais: **Ingestão** (offline/background) e **Consulta** (online/tempo real).

```text
[PDFs Públicos] ──► Extração de Texto ──► Chunking ──► Embeddings ──► Banco Vetorial (FAISS)
                                                                            │
                                                                            ▼
[Usuário] ──► Pergunta ──► Busca por Similaridade (Top-K) ──────────────────┤
                                                                            │
[Usuário] ◄── Resposta Stream + Citação ◄── LLM (Prompt + Contexto) ◄──────┘
Ingestão de Dados:Extração: Leitura de documentos PDF via pdfplumber/PyPDF.Chunking: Divisão do texto em blocos (chunks) com sobreposição (overlap) para preservar o contexto semântico.Vetorização: Geração de embeddings para cada bloco.Indexação: Armazenamento dos vetores e metadados no FAISS.Consulta & Recuperação:Similarity Search: A pergunta do usuário é convertida em vetor e o FAISS retorna os K trechos mais relevantes.Prompt Augmentation: Os trechos recuperados são injetados no contexto do prompt enviado ao LLM.Geração Fundamentada: O LLM sintetiza a resposta e cita os documentos/artigos de origem.🛠️ Stack TecnológicaComponenteTecnologiaDescriçãoLinguagemPython 3.10+Ecossistema base do projetoAPI WebFastAPI + UvicornServer backend assíncrono e de alta performanceBanco VetorialFAISSFacebook AI Similarity Search para busca de vetores em memória/discoProcessamento PDFpdfplumber / PyPDFExtração precisa de texto e metadados dos PDFsEmbeddings & LLMOpenAI / Anthropic APIModelo de linguagem para síntese e vetorizaçãoStreamingServer-Sent Events (SSE)Envio de tokens em tempo real para a interface🚀 Como Rodar o ProjetoPré-requisitosPython 3.10 ou superiorChave de API de um provedor de LLM (OpenAI ou Anthropic)1. Clonar o repositório e criar ambiente virtualBashgit clone [https://github.com/seu-usuario/rag-documentos-publicos.git](https://github.com/seu-usuario/rag-documentos-publicos.git)
cd rag-documentos-publicos

python -m venv .venv
# Linux/macOS:
source .venv/bin/activate
# Windows:
.venv\Scripts\activate
2. Instalar dependênciasBashpip install -r requirements.txt
3. Configurar variáveis de ambienteCrie um arquivo .env na raiz do projeto com as suas credenciais:Snippet de código# Provedor e Chaves de API
OPENAI_API_KEY=sua_chave_aqui_opcional
ANTHROPIC_API_KEY=sua_chave_aqui_opcional

# Configurações do RAG
EMBEDDING_MODEL=text-embedding-3-small
CHUNK_SIZE=1000
CHUNK_OVERLAP=150
TOP_K_RESULTS=4
4. Executar a aplicaçãoBashuvicorn app.main:app --reload
Acesse a documentação interativa da API no navegador:Swagger UI: http://127.0.0.1:8000/docsReDoc: http://127.0.0.1:8000/redoc📡 Endpoints PrincipaisPOST /perguntarRealiza uma consulta em linguagem natural à base de documentos.Exemplo de Payload:JSON{
  "pergunta": "Qual o prazo para interposição de recurso no edital de estágio?",
  "stream": true
}
Exemplo de Resposta (JSON não-streaming):JSON{
  "resposta": "O prazo para interposição de recursos é de 2 (dois) dias úteis a contar da publicação do resultado preliminar (Art. 15, § 1º).",
  "fontes": [
    {
      "documento": "Edital_Estagio_2026.pdf",
      "pagina": 12,
      "trecho_extraido": "Art. 15, § 1º - O candidato poderá interpor recurso no prazo de 2 dias úteis..."
    }
  ]
}
📌 Roadmap de Desenvolvimento[x] Definição da arquitetura básica e stack[ ] Pipeline de ingestão e parsing de PDFs[ ] Implementação da estratégia de chunking dinâmico[ ] Indexação e busca vetorial com FAISS[ ] Endpoint /perguntar (resposta estática)[ ] Suporte a respostas em streaming via Server-Sent Events (SSE)[ ] Módulo de avaliação e testes com corpus real de documentos públicos (ex: MPRN)[ ] Interface gráfica simples (Streamlit ou Web UI)Nota de Responsabilidade: Este projeto é voltado para auxílio e consulta ágil. As respostas geradas não substituem a leitura oficial do Diário Oficial ou das publicações institucionais vigentes.
