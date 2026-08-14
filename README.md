# Smart Event Planner 

**Program:** Building Agentic AI Systems by SDAIA Academy

**Session Dates:** 9th of August, 2026 – 13th of August, 2026

**Declared Track:** Track A

## Team Members
- Setah Mohammed Alajmi
- Raneem Abdullah Alsheddi
- Jana Hamad Alhumaizi
- Shatha Hamad Bin Mana
- Sarah Abdulaziz

---

## Project Description

**Smart Event Planner** is an agentic AI system for planning events end-to-end. A user
describes an event in natural language (type, guest count, budget, date, indoor/outdoor
preference, theme), and the system extracts a structured request, plans the event using
real tools, answers grounded questions from a knowledge base, and routes between planning
and knowledge specialists automatically — all with a human approval step before anything
is finalized.

## Capabilities
- **Structured intake** — natural-language requests are parsed into a validated Pydantic
  `EventRequest` (no invented fields).
- **Real planning tools** — budget allocation, venue search, catering search, decoration
  suggestions, and a preparation checklist, each driven by the arguments the LLM provides.
- **Multi-agent routing** — an LLM supervisor delegates each request to either the
  `planning_agent` or the `knowledge_agent`, with no keyword-based routing.
- **Retrieval-Augmented Generation (RAG)** — event-planning guidance (venue selection,
  safety, graduation-event planning) is loaded from PDFs, chunked, embedded, and retrieved
  to ground the knowledge agent's answers.
- **Context & state management** — short-term workflow state via a LangGraph
  checkpointer/`thread_id`, and long-term user preferences via a separate `Store`, with a
  cross-thread test proving the memory persists across threads.
- **Human-in-the-loop** — the workflow pauses with `interrupt()` before finalizing a plan
  and resumes with `Command(resume=...)` once a human approves, rejects, or edits it.
- **Reliability** — core operations are wrapped in the LangGraph Functional API
  (`@task` / `@entrypoint`) with a real `RetryPolicy` and a fallback strategy for failed or
  empty results.
- **Observability** — LangSmith tracing is enabled end-to-end, with a results-based
  write-up generated from a real trace query (slowest span, error count, tool activity).

**Note:** this project is implemented as **four separate notebooks, not one**. See
`notebooks/` below.

---

## Repository Structure

```
Building-Agentic-AI-Systems-by-SDAIA-Academy/
├── README.md
├── .gitignore
└── notebooks/
    ├── 01_agent_fundamentals.ipynb
    ├── 02_rag_and_multi_agent_routing.ipynb
    ├── 03_context_state_and_human_in_the_loop.ipynb
    └── 04_functional_api_reliability_and_observability.ipynb
```

| Notebook | Covers |
|---|---|
| `01_agent_fundamentals.ipynb` | Agent Fundamentals |
| `02_rag_and_multi_agent_routing.ipynb` | RAG Pipeline · Multi-Agent / Routing Architecture · Workflow Pattern |
| `03_context_state_and_human_in_the_loop.ipynb` | Context & State Management · Human-in-the-Loop |
| `04_functional_api_reliability_and_observability.ipynb` | LangGraph Functional API & Error Handling · LangSmith Observability |

---

## How to Run

These notebooks were built and run in **Google Colab**.

1. Add `GROQ_API_KEY` and `LANGSMITH_API_KEY` to Colab Secrets (key icon in the left
   sidebar) and enable notebook access for both. No API keys are hardcoded anywhere —
   they're loaded exclusively via `google.colab.userdata`.
2. Open a notebook in Colab and use **Runtime → Run all**. Each notebook installs its own
   dependencies in its first code cell(s).
3. Each notebook is self-contained and can be run on its own, in any order — where a
   notebook depends on tools defined in `01_agent_fundamentals.ipynb`, those definitions
   are re-declared inside it.

### Working with the four notebooks

- **Open and run each one separately.** They are not chained — running `01` does not set
  anything up for `02`, `03`, or `04`. Each notebook has its own `pip install` and its own
  Colab Secrets loading cell at the top, so opening any single notebook and running
  **Runtime → Run all** is enough to fully execute it, without needing to run the others
  first.
- **No shared state between notebooks.** Each notebook runs in its own separate Colab
  runtime. In-memory objects — the vector store, the LangGraph checkpointer, the
  long-term `Store` — exist only for the lifetime of that notebook's session. For
  example, the cross-thread memory test in `03` reads a preference that was written
  earlier in that same notebook's run, not from `01` or `02`.
- **Add secrets in every notebook you plan to run.** Since each is a separate runtime,
  Colab Secrets need notebook access enabled per notebook (or per Colab session, if
  you're re-using one) — it's not a one-time setup that carries over.
- **Suggested reading order (not a required run order):** `01` → `02` → `03` → `04`.
  This follows the flow of the system (build the tools first, then routing/RAG, then
  state/approval, then reliability/observability), but since nothing is shared, they can
  be opened and run in any order or independently by different reviewers at the same
  time.
- **Editing one doesn't affect the others.** Since the notebooks don't import from each
  other, a change made inside `04_functional_api_reliability_and_observability.ipynb` (for
  example) only needs that one file re-saved and pushed — no need to touch `01`–`03`.

---

## Technical Documentation

### Architecture

```
User request
     │
     ▼
Structured intake (Pydantic EventRequest)
     │
     ▼
Supervisor (LLM-based routing)
     ├──► planning_agent  → budget / venue / catering / decoration / checklist tools
     └──► knowledge_agent → RAG retriever over event-knowledge PDFs
     │
     ▼
LangGraph workflow: build context → generate plan → save preferences (long-term Store)
     → human_approval (interrupt) → [pause for human] → finalize_plan (resume)
     │
     ▼
Functional API layer (@task / @entrypoint): retries + fallback for budget/venue steps
     │
     ▼
LangSmith tracing throughout
```

### Design choices
- **RAG design:** 2-Step RAG (retrieve, then generate) rather than Agentic RAG — event
  knowledge questions should always be grounded in the project's documents rather than
  leaving retrieval up to agent discretion, and the knowledge base is small enough that
  a Hybrid design would add routing complexity without benefit.
- **Workflow pattern:** Routing — requests naturally split into two specialist
  responsibilities (operational planning vs. document-grounded knowledge), and the LLM
  supervisor decides which one applies per request.
- **Reliability:** a formal `RetryPolicy` (attempts, backoff) handles transient failures
  in tool tasks; a separate fallback function supplies a safe default when a search
  returns no results or an exception occurs.

---

## Git Practices

- `.gitignore` excludes secrets (`.env`, key files), Python/Jupyter build artifacts,
  runtime-generated files (the `event_knowledge/` PDFs created at run time), and local
  vector store / cache directories.
- No API keys are committed anywhere in this repository — all secrets are loaded from
  Colab Secrets at runtime.
- Commits are scoped and incremental rather than a single bulk upload.

---

## Acknowledgements

This project was completed as the capstone for the **Building Agentic AI Systems**
program run by **SDAIA Academy**.
[SDAIA Academy's GitHub](https://github.com/SDAIAAcademy)
