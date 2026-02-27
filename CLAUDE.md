# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with this repository.

## Running the App

**1. Install dependencies** (first time only):
```bash
uv sync
```

**2. Add your API key** — create `.env` in the repo root:
```
ANTHROPIC_API_KEY=your_key_here
```

**3. Start the server:**
```bash
./run.sh
# or manually:
cd backend && uv run uvicorn app:app --reload --port 8000
```

- **UI:** http://localhost:8000
- **API docs:** http://localhost:8000/docs

No tests, linting, or CI exist in this project.

---

## Project Layout

| Directory | Purpose |
|-----------|---------|
| `backend/` | FastAPI app — all server-side logic |
| `frontend/` | Vanilla HTML/JS/CSS — served as static files by FastAPI |
| `docs/` | Course `.txt` files — auto-loaded into ChromaDB at startup |

---

## Architecture

A RAG chatbot: Claude uses a search tool to look up course content before answering. Two API endpoints: `POST /api/query` and `GET /api/courses`.

### Query Flow

| Step | What happens | File |
|------|-------------|------|
| 1 | User submits a question; JS posts `{query, session_id}` | `frontend/script.js` |
| 2 | Request validated; session created if new | `backend/app.py` |
| 3 | Conversation history loaded; AI called with search tool | `backend/rag_system.py` |
| 4 | **1st Claude call** — Claude decides whether to search | `backend/ai_generator.py` |
| 5 | If searching: ChromaDB queried for matching chunks | `backend/search_tools.py` → `vector_store.py` |
| 6 | **2nd Claude call** — search results sent back; Claude writes final answer | `backend/ai_generator.py` |
| 7 | Sources + answer returned; session history updated | `backend/rag_system.py` |

### ChromaDB Collections

Two collections in `backend/chroma_db/`:
- **`course_catalog`** — course metadata; used to fuzzy-match course names via embedding search
- **`course_content`** — chunked lesson text; used for semantic search

### Session History

Conversation history is injected into Claude's **system prompt** as plain text — not as structured messages. Max 2 exchanges are retained (configurable via `MAX_HISTORY` in `backend/config.py`).

### Course Document Format

Files in `docs/` must follow this structure for the parser to work:

```
Course Title: ...
Course Link: ...
Course Instructor: ...

Lesson 0: ...
Lesson Link: ...
[lesson content]
```

Parser logic: `backend/document_processor.py`

### Config

All tunable settings (model, chunk size, max results) are in the `Config` dataclass in `backend/config.py`.
