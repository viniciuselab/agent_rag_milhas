# Agente RAG - Milhas Aéreas

Projeto de estudo de **Engenharia de IA** que implementa um agente de perguntas e respostas com **RAG (Retrieval-Augmented Generation)** sobre documentos de cartões de crédito com benefícios de milhas e viagens.

Em vez de depender apenas do conhecimento do modelo, o agente busca os trechos mais relevantes nos PDFs e responde **somente com base nesse conteúdo**, o que reduz alucinações. A inferência roda localmente com o Ollama, sem chamadas a APIs pagas.

## Como funciona

```
PDFs ──► Carregamento ──► Chunking ──► Embeddings ──► FAISS
                                                        │
Pergunta ──────────────────────────► Retriever ◄────────┘
                                         │
                                  Prompt + contexto
                                         │
                                    LLM (Gemma 3)
                                         │
                                      Resposta
```

1. **Carregamento:** os PDFs da pasta `documentos/` (cartões GTB Gold, Platinum e Standard, edição nov/2023) são lidos com o `PyPDFLoader`. Cada PDF vira um `Document` com metadados (`cartao`, `source`, `total_pages`).
2. **Chunking:** o texto é dividido com o `RecursiveCharacterTextSplitter`, usando o tokenizer do `BAAI/bge-m3` para medir o tamanho (chunks de 1250 tokens, com overlap de 150).
3. **Embeddings e indexação:** os chunks são vetorizados com o modelo `bge-m3` (via Ollama) e armazenados em um índice **FAISS**.
4. **Recuperação:** o retriever busca no índice os trechos mais próximos da pergunta.
5. **Geração:** um prompt instrui o LLM a responder usando exclusivamente o contexto recuperado, em no máximo 100 palavras. O modelo usado é o `gemma3:1b`, também via Ollama.

A cadeia final é montada com LCEL: `prompt | model_llm | StrOutputParser()`.

## Tecnologias

| Área | Ferramenta |
|---|---|
| Linguagem | Python 3 |
| Ambiente | Jupyter Notebook |
| Orquestração | LangChain (`langchain`, `langchain-community`, `langchain-ollama`, `langchain-text-splitters`) |
| Leitura de PDF | `pypdf`, `unstructured[pdf]` |
| Tokenização | Hugging Face `transformers` (`BAAI/bge-m3`) |
| Embeddings | `bge-m3` via Ollama |
| Banco vetorial | FAISS (`faiss-cpu`) |
| LLM | `gemma3:1b` via Ollama |



