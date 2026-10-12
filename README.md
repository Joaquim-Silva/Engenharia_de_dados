    # 📘 Assistente Corporativo de RH com RAG

> **Projeto 1 — Busca Semantica em Documentos**  
> Sistema de Perguntas e Respostas com RAG sobre Documentos Internos de RH

---

## 📋 Índice

- [Visão Geral](#-visão-geral)
- [Objetivo](#-objetivo)
- [Arquitetura do Projeto](#️-arquitetura-do-projeto)
- [Fluxo de Funcionamento](#-fluxo-de-funcionamento)
- [Componentes Técnicos](#-componentes-técnicos)
- [Instalação e Execução](#️-instalação-e-execução)
- [Exemplos de Teste](#-exemplos-de-teste)
- [Stack Tecnológica](#-stack-tecnológica)
- [Diferenciais do Projeto](#-diferenciais-do-projeto)
- [Observações Técnicas](#-observações-técnicas)
- [Autor](#-autor)

---

## 🧠 Visão Geral

Este projeto implementa um **Assistente Corporativo de Perguntas e Respostas** baseado na arquitetura **RAG (Retrieval-Augmented Generation)**. O sistema responde perguntas em linguagem natural sobre **políticas internas de Recursos Humanos**, consultando uma base de conhecimento composta por documentos PDF previamente indexados.

A aplicação foi construída com **Streamlit** (interface web), **LangChain** (orquestração do pipeline RAG), **ChromaDB** (banco de dados vetorial), **HuggingFace Embeddings** (vetorização) e **Groq** (LLM de alta performance).

---

## 🎯 Objetivo

Criar um assistente inteligente capaz de:

- 📄 Ler e indexar políticas internas de RH (PDFs);
- 🔍 Realizar **busca semântica** sobre o conteúdo indexado;
- 💬 Gerar **respostas contextualizadas** com base nos trechos recuperados;
- 📚 Exibir **metadados das fontes** (arquivo e página) utilizadas na resposta;
- ⚙️ Permitir **gestão dinâmica** da base de conhecimento (upload e limpeza).

---

## 🏗️ Arquitetura do Projeto

```
📁 projeto/
│
├── app_buscasemantica.py      # Interface Streamlit (frontend + orquestração)
├── rag_engine.py              # Motor RAG (embeddings, vetorização, LLM, chain)
├── requirements.txt           # Dependências do projeto
├── LEIAME.txt                 # Instruções de instalação e execução
├── politicas_rh.pdf           # Documento de exemplo (base de conhecimento)
└── chroma_db_data/            # Banco vetorial persistente (gerado automaticamente)
```

---

## 🔄 Fluxo de Funcionamento

### 1️⃣ Ingestão de Documentos

- O usuário faz **upload de PDFs** pela sidebar;
- Os arquivos são salvos temporariamente;
- O `PyPDFDirectoryLoader` lê os PDFs;
- O `RecursiveCharacterTextSplitter` divide o texto em **chunks** de 1000 caracteres com overlap de 200;
- Os chunks são vetorizados pelo modelo **`all-MiniLM-L6-v2`** (HuggingFace);
- Os vetores + metadados são persistidos no **ChromaDB** (coleção `rh_policies`).

### 2️⃣ Consulta (Query)

- O usuário digita uma pergunta no chat;
- O `RetrievalQA` realiza **busca semântica** no ChromaDB (top-3 chunks mais similares);
- Os chunks recuperados são injetados no prompt do LLM (**Groq — `openai/gpt-oss-120b`**);
- O LLM gera uma resposta **fundamentada apenas no contexto recuperado**;
- A resposta é exibida junto com os **metadados das fontes** (arquivo + página + trecho).

### 3️⃣ Gestão da Base

- Botão **"Limpar Banco de Dados"** deleta a coleção e recria uma vazia.

---

## 🧩 Componentes Técnicos

### 🔹 `rag_engine.py` — Motor RAG

| Componente | Tecnologia | Função |
|-----------|-----------|--------|
| **Embeddings** | `HuggingFaceEmbeddings` (`all-MiniLM-L6-v2`) | Converte texto em vetores |
| **LLM** | `ChatGroq` (`openai/gpt-oss-120b`) | Gera respostas contextualizadas |
| **Vector Store** | `Chroma` | Armazena e busca vetores |
| **Loader** | `PyPDFDirectoryLoader` | Lê PDFs de um diretório |
| **Splitter** | `RecursiveCharacterTextSplitter` | Divide texto em chunks |
| **Chain** | `RetrievalQA` (`chain_type="stuff"`) | Orquestra retrieval + geração |

**Métodos principais:**

```python
ingest_documents(temp_dir_path)  # Processa e indexa PDFs
get_response(query)              # Executa busca + geração
clear_database()                 # Limpa a coleção vetorial
```

### 🔹 `app_buscasemantica.py` — Interface Streamlit

**Sidebar:**
- Upload múltiplo de PDFs;
- Botão "Processar Documentos";
- Botão "Limpar Banco de Dados";
- Bloco de contato (suporte).

**Área principal:**
- Chat com histórico persistente (`st.session_state`);
- Exibição de fontes consultadas em `st.expander`;
- Spinner durante consultas.

---

## ⚙️ Instalação e Execução

### Pré-requisitos

- **Conda** (recomendado) ou Python 3.13+
- Chave de API do **Groq** (configurada em `.env`)

### Passo a Passo

```bash
# 1. Criar ambiente virtual
conda create --name dsamebvp1 python=3.13

# 2. Ativar ambiente
conda activate dsamebvp1

# 3. Instalar dependências
conda install pip
pip install -r requirements.txt

# 4. Configurar variável de ambiente (.env)
echo "GROQ_API_KEY=sua_chave_aqui" > .env
echo "GROQ_MODEL=openai/gpt-oss-120b" >> .env

# 5. Executar a aplicação
streamlit run app_buscasemantica.py
```

### Limpeza do Ambiente (Opcional)

```bash
conda deactivate
conda remove --name dsamebvp1 --all
```

---

## 🧪 Exemplos de Teste

Após indexar o `politicas_rh.pdf`, faça as seguintes perguntas:

| # | Pergunta |
|---|----------|
| 1 | Qual a antecedência mínima para solicitação de férias? |
| 2 | Há necessidade de presença física em algum momento no modelo de trabalho remoto? |
| 3 | Posso instalar softwares por conta própria no computador da empresa? |

---

## 📊 Stack Tecnológica

| Camada | Tecnologia |
|--------|-----------|
| **Interface** | Streamlit 1.52 |
| **Orquestração** | LangChain 1.0 |
| **LLM** | Groq (GPT-OSS 120B) |
| **Embeddings** | Sentence-Transformers (MiniLM-L6-v2) |
| **Vector DB** | ChromaDB 1.3 |
| **Parsing PDF** | pypdf 6.4 |
| **Ambiente** | Python 3.13 + Conda |

---

## ✅ Diferenciais do Projeto

- 🎯 **RAG completo e funcional** com pipeline de ingestão, retrieval e geração;
- 💾 **Persistência vetorial** (não precisa reindexar a cada execução);
- 🔎 **Transparência** — exibe fontes e trechos utilizados;
- ⚙️ **Gestão dinâmica** — upload e limpeza via interface;
- ⚡ **LLM de última geração** via Groq (baixa latência);
- 💬 **Interface amigável** com chat e histórico.

---

## 📌 Observações Técnicas

- O modelo `all-MiniLM-L6-v2` é leve (~80MB) e otimizado para busca semântica;
- O `chain_type="stuff"` insere todos os chunks recuperados diretamente no prompt (ideal para contextos pequenos);
- O parâmetro `k=3` define quantos chunks são recuperados — pode ser ajustado conforme necessidade;
- O ChromaDB persiste em `./chroma_db_data/` — basta deletar a pasta para resetar manualmente.

---

## 👨‍💻 Autor

**Projeto 1 — Busca Semantica em Documentos**  
*Assistente Corporativo de RH com RAG sobre Documentos Internos*

📧 Suporte: [joaquim.diego8@gmail.](mailto:joaquim.diego8@gmail.com)

---

<p align="center">
  <b>⭐ Se este projeto foi útil, considere dar uma estrela! ⭐</b>
</p>