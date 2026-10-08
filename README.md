# Graph-RAG

A GraphRAG demo with LangChain, Groq and Neo4j. The notebook [Graph_RAG_Neo4j.ipynb](Graph_RAG_Neo4j.ipynb) walks through the full pipeline:

**Documents → Chunking → Graph Extraction (LLM) → Neo4j Storage → Graph Retrieval (Cypher) → LLM Answer**

It extracts `Person`, `Movie` and `Genre` nodes from a small movie dataset and stores them in Neo4j. It then answers questions such as *"Which movies did Christopher Nolan direct?"* with `GraphCypherQAChain`.

## Prerequisites

- Python 3.10+ (tested on 3.13)
- A **Groq API key**: create one at <https://console.groq.com/keys>
- A **Neo4j database**: a free [Neo4j AuraDB](https://neo4j.com/cloud/aura-free/) instance works. When you create the instance, save the credentials file it gives you. It contains the URI, username and password.

## Setup

### 1. Clone the repository

```bash
git clone <repo-url>
cd Graph-RAG
```

### 2. Create and activate a virtual environment

```bash
python -m venv grrag
```

Windows (PowerShell):

```powershell
.\grrag\Scripts\Activate.ps1
```

macOS / Linux:

```bash
source grrag/bin/activate
```

### 3. Install dependencies

```bash
pip install langchain langchain-community langchain-experimental langchain-groq langchain-neo4j neo4j python-dotenv ipykernel
```

### 4. Create the `.env` file

Create a file named `.env` in the project root, next to the notebook, with these keys:

```env
# Groq (LLM): https://console.groq.com/keys
GROQ_API_KEY=your_groq_api_key

# Neo4j (from your AuraDB credentials file)
NEO4J_URI=neo4j+s://<your-instance-id>.databases.neo4j.io
NEO4J_USERNAME=<your-username>
NEO4J_PASSWORD=<your-password>
NEO4J_DATABASE=<your-database-name>
```

| Key | Where to get it |
| --- | --- |
| `GROQ_API_KEY` | Groq Console → API Keys → *Create API Key* |
| `NEO4J_URI` | AuraDB credentials file / instance details page |
| `NEO4J_USERNAME` | AuraDB credentials file |
| `NEO4J_PASSWORD` | AuraDB credentials file (shown only once at creation) |
| `NEO4J_DATABASE` | AuraDB credentials file (usually the instance ID, or `neo4j`) |

Notes:
- Don't put spaces around `=` or quotes around the values.
- `.env` is listed in `.gitignore`. **Never commit it.**
- The notebook loads the file with `load_dotenv()`. If you edit `.env` after the kernel has started, restart the kernel.

## Running the notebook

1. Open `Graph_RAG_Neo4j.ipynb` in VS Code or Jupyter and select the `grrag` interpreter or kernel.
2. Run the cells from top to bottom:
   1. **Connect to Neo4j**: loads `.env` and creates a `Neo4jGraph`.
   2. **Create the LLM**: `ChatGroq` with `openai/gpt-oss-120b`.
   3. **Dataset and chunking**: three short movie documents, split with `RecursiveCharacterTextSplitter`.
   4. **Graph extraction**: `LLMGraphTransformer` turns the chunks into nodes and relationships.
   5. **Store in Neo4j**: `graph.add_graph_documents(...)`.
   6. **Retrieval**: runs Cypher queries directly against the graph.
   7. **Generation**: `GraphCypherQAChain` turns each question into Cypher, runs it, and has the LLM answer from the results.

To see the graph, open Neo4j Aura → *Query* and run `MATCH (n) RETURN n`.

## Troubleshooting

| Error | Fix |
| --- | --- |
| `GroqError: The api_key client option must be set...` | `GROQ_API_KEY` isn't loaded. Check that `.env` is in the project root, that `load_dotenv()` runs before `ChatGroq`, and restart the kernel. |
| `ServiceUnavailable` / `AuthError` from Neo4j | Check the `NEO4J_*` values. Also check that the Aura instance is running, since free instances pause when idle. |
| Re-running step 6 creates duplicate nodes | Clear the graph with `graph.query("MATCH (n) DETACH DELETE n")` before reloading. |
