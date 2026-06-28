# Total Recall — Agent Harness Workshop

[![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://github.com/codespaces/new?repo=1280760538&ref=main&machine=standardLinux32gb&location=WestUs2&devcontainer_path=.devcontainer/devcontainer.json)

Build a **self-improving agent harness** from the ground up on a single **Oracle AI Database** — and
watch the same harness running in your browser as you build it. The agent persists its own memory,
grounds itself in your schema's meaning, turns the work it does into reusable **skills** and scheduled
**automations**, and keeps its context window flat — all inside the database.

> `Agent = Model + Harness`. The model is a frozen reasoning utility; the **harness** is everything
> around it that makes it useful — and everything that makes it *get better*: **memory** (episodic /
> semantic / procedural), **retrieval** (vector + keyword + rerank), **tools**, **identity**, and a
> **loop** that ties them together. This workshop builds all of it on one converged database — no
> separate vector DB, key-value store, or queue.

![The harness, dissected layer by layer — the model is layer 1; everything else is what you build](images/agent_harness.png)

The workshop ships a **complete** notebook (the runnable reference) and a **student** notebook with
**19 TODOs** to fill in (a full ~1–1.5 hour session), each paired with a `docs/todoN.md` explainer
and a copy-paste solution. A
**Codespace** brings up Oracle AI Database, provisions it, and auto-starts the **appbook** so you see
the finished harness running while you rebuild it.

## What's inside

```
total_recall_complete.ipynb    the full, runnable build (Parts 0–10) — the answer key
total_recall_student.ipynb     the same notebook with 19 TODOs to fill in
docs/todo1.md … todo19.md      a guided explainer + copy-paste solution for each TODO
workshop_setup.py              the plumbing the notebook imports (config + connection + SQL helpers)
app/                           the appbook — the harness served as a click-through browser app
scripts/
  seed_oracle.py               headless provisioner (AGENT user + in-DB ONNX embedder)
  build_student_notebook.py    regenerates the student notebook + docs from the complete notebook
.devcontainer/                 one-click Codespaces: Oracle AI Database + the app, ready on arrival
frequently_asked_question.md   common Codespaces pitfalls (right kernel · wait for the build) + fixes
```

### The notebook — build the harness
`total_recall_complete.ipynb` builds the agent bottom-up in Parts 0–10: a least-privilege `AGENT`
schema, **in-database ONNX embeddings**, an in-DB scratch filesystem, the retrieval ladder (keyword +
vector + **RRF** + cross-encoder rerank), **OAMP** cognitive memory with the **context card**, a
schema **semantic layer**, HNSW-searched **tool** and **skill** registries, scheduled **automations**,
a typed **LangGraph** agent loop made durable by the `langgraph-oracledb` `OracleSaver` checkpointer,
and **context engineering** that keeps the window flat.

![Oracle is the engine, not a step — ingestion, embeddings, vector search and rerank all run in the database](images/oracle_engine.png)

### The appbook — see it running
`app/` is the same harness served as an **appbook**: a working agent you click through in the browser,
organised by harness layer (foundation → memory substrate → retrieval → cognitive memory → semantic
layer → skills & automations → agent loop → context engineering → mission control). It's a **FastAPI
backend + a dependency-free JavaScript SPA** (no build step), streaming over SSE — everything except the chat model runs inside
the database — and the chat model itself runs on **Oracle's OCI Generative AI**, so the whole stack is Oracle. See [`app/README.md`](app/README.md).

> The notebook and the appbook share **one** `AGENT` schema, so the app you click through is literally
> the harness the notebook builds.

## Run it in GitHub Codespaces (recommended)

**One click:** [![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://github.com/codespaces/new?repo=1280760538&ref=main&machine=standardLinux32gb&location=WestUs2&devcontainer_path=.devcontainer/devcontainer.json)  — or follow the steps below.

1. **Code ▸ Create codespace.** The dev container brings up Oracle AI Database, installs the deps, and
   **provisions the database** — it creates the `AGENT` user and loads the 384-dim in-database ONNX
   embedder. Set your **`OCI_GENAI_API_KEY`** as a Codespaces secret (org-level recommended) so the
   agent can call the model.
2. When setup finishes, the **appbook opens automatically on port 8000** — the sidebar status badge
   turns green as the harness warms.
3. Open **`total_recall_student.ipynb`** and pick the Python 3.12 kernel. Work the 19 TODOs top to
   bottom; `total_recall_complete.ipynb` next to it is the answer key.

> **Hit a snag?** The two things that trip people up — **selecting the right kernel** and **running a
> cell before the build finishes** — are covered with screenshots in
> [`frequently_asked_question.md`](frequently_asked_question.md).

> Provisioning is **idempotent** and the database persists in a named volume — rebuilding a Codespace
> is fast, and the loaded ONNX model is never re-downloaded once cached.

## Run it locally

Prereqs: **Docker**, a **Python 3.11+** env, and an **OCI Generative AI API key** (`OCI_GENAI_API_KEY`).

```bash
# 1. Oracle AI Database (the whole harness runs here)
docker run -d --name oracle-free -p 1521:1521 -e ORACLE_PASSWORD=OraclePwd_2025 \
  gvenzl/oracle-free:latest

# 2. install deps + provision the database (creates AGENT, loads the in-DB embedder)
pip install -r app/requirements.txt jupyterlab ipykernel \
  langgraph langgraph-oracledb langchain langchain-openai openai matplotlib pandas onnx
export ORA_DSN=localhost:1521/FREEPDB1 ORA_ADMIN_PWD=OraclePwd_2025 ORA_AGENT_PWD=AgentPw_2026
export LLM_PROVIDER=oci OCI_GENAI_API_KEY=...
python scripts/seed_oracle.py

# 3. run the appbook …
cd app && ./run.sh                 # → http://127.0.0.1:8000
# 4. … or open the notebook
jupyter lab                        # total_recall_student.ipynb
```

> **Seeing the diagrams.** The notebook embeds architecture diagrams as ` ```mermaid ` blocks. They
> render automatically in a Codespace and in **JupyterLab 4.1+**. In **VS Code**, install the
> _Markdown Preview Mermaid Support_ extension (`bierner.markdown-mermaid`) and reload. The agent-graph
> diagram is a separate `draw_mermaid_png()` cell — it renders a PNG when you **run** it (needs
> internet; it calls the mermaid.ink renderer).

## The TODOs

The **student** notebook blanks nineteen implementations into TODOs — each paired with a
`docs/todoN.md` explainer and a **check cell that must pass** before you continue. They walk the
harness bottom-up, one primitive at a time:

| #  | Build                                               | Harness layer        |
| -- | --------------------------------------------------- | -------------------- |
| 1  | Talk to the **reasoning core** (the bare model)     | model                |
| 2  | Design the **scratch table** (SecureFile LOB)       | memory substrate     |
| 3  | The in-database **scratch filesystem** (write/read) | memory substrate     |
| 4  | **Grep** the agent's scratch memory                 | memory substrate     |
| 5  | **In-database embeddings** for the vector store     | encoding             |
| 6  | Choose the **vector distance** strategy             | encoding / retrieval |
| 7  | **Chunk** text into overlapping windows             | encoding             |
| 8  | **Vector** (semantic) retrieval                     | retrieval            |
| 9  | Fuse keyword + vector with **RRF**                  | retrieval            |
| 10 | **Rerank** the shortlist (cross-encoder)            | retrieval            |
| 11 | Embed text **inside** the database (OAMP embedder)  | encoding / memory    |
| 12 | **Promote** scratch files → long-term memory        | continual learning   |
| 13 | Make tools **retrievable by meaning** (HNSW)        | tools                |
| 14 | A safe, read-only **SQL tool**                      | tools                |
| 15 | **Save** a SHA-versioned skill                      | memory (procedural)  |
| 16 | **Skills** as searchable memory                     | memory (procedural)  |
| 17 | **Promote a workflow** into a skill                 | continual learning   |
| 18 | **Harvest** recurring workflows into skills         | continual learning   |
| 19 | **The agent loop**                                  | the whole harness    |

Regenerate the student notebook + docs from the complete notebook any time:

```bash
python scripts/build_student_notebook.py
```

## Requirements

- An **OCI Generative AI API key** (`OCI_GENAI_API_KEY`) — the only outbound call. The LLM runs on
  Oracle too (OCI GenAI's OpenAI-compatible endpoint); default model `xai.grok-4-1-fast-reasoning`, set
  `LLM_MODEL` to change it (or `LLM_PROVIDER=openai` + `OPENAI_API_KEY` to use OpenAI instead).
- **Docker** for Oracle AI Database (the Free image — no licence, no cloud account).
- **No GPU and no PyTorch** — embeddings run as an ONNX model **inside Oracle**.

> **Note on API keys.** This workshop is wired with **four OCI Generative AI API keys**
> (`OCI_GENAI_API_KEY` plus `OCI_GENAI_API_KEY_2`–`_4`) for access to Grok. Each notebook kernel and
> appbook process picks a random starting key and **automatically rotates to the next key** on any
> rate-limit, auth, or transient error — so a large cohort spreads load across the keys and a single
> bad key never blocks you.