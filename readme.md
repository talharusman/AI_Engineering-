# Rubik's Cube Race

**Student:** talha Rusman  
**Roll No:** 23k-0065  
**Course:** AI Engineering, Assignment 02

## Overview

This project is a FastAPI web application that benchmarks four AI model slots on the same Rubik's Cube task. A cube is represented by a deterministic 54-facelet state. Each model chooses one legal move per turn; the dashboard shows the cube, moves, timing, results, cache use, and aggregate statistics.

The game and benchmark behaviour has not been changed during the folder cleanup. Only file locations and the imports/path configuration needed to support those locations were updated.

## Project structure

```text
AI-engineering-main/
├── backend/                 # Python API, cube engine, model adapters, scripts
│   ├── main.py              # FastAPI application and benchmark orchestration
│   ├── cube.py              # Cube state, legal moves, BFS optimum, scramble generator
│   ├── llm.py               # Mock, Ollama, and OpenRouter adapters; cache and budget logic
│   ├── test_cube.py         # Deterministic engine tests
│   ├── dry_check.py         # Prints API payloads without making requests
│   ├── report.py            # Builds Markdown statistics from results.json
│   └── requirements.txt     # Python dependencies
├── frontend/
│   └── index.html           # Single-page dashboard (HTML, CSS, and JavaScript)
├── .env.example             # Environment-variable template
├── cache.json               # Persistent API response cache and OpenRouter call counters
├── results.json             # Recorded benchmark outcomes
├── requests.log             # Log of real API requests
├── report_ollama.md         # Existing generated Ollama report
├── report_openrouter.md     # Existing generated OpenRouter report
└── demo1.mp4, demo2.mp4     # Demo recordings
```

## How it works

- `cube.py` implements all 18 standard face moves and uses breadth-first search to select a reproducible starting state exactly three moves from solved.
- `main.py` runs every model against that same starting state, with a normal limit of 20 moves. It exposes `/api/run`, `/api/state`, `/api/budget`, `/api/config`, and `/api/stats`.
- `llm.py` supports three modes:
  - `mock`: simulated responses; no API call.
  - `ollama`: requests a locally running Ollama server.
  - `openrouter`: makes capped real API requests using `OPENROUTER_API_KEY`.
- The server serves `frontend/index.html` at the site root. The frontend reads the configured student information from `/api/config`; its defaults are **talha Rusman** and **23k-0065**.

## Setup and run

Use Python 3.10+ and run these commands from the repository root:

```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
pip install -r backend\requirements.txt
Copy-Item .env.example .env
uvicorn backend.main:app
```

Open http://127.0.0.1:8000 in a browser. Start with **Mock** mode to confirm the application works without credentials.

For Ollama, ensure the local service is running. For OpenRouter, edit `.env` and set a valid key:

```env
OPENROUTER_API_KEY=your_key_here
STUDENT_NAME=talha Rusman
STUDENT_ROLL=23k-0065
OLLAMA_URL=http://localhost:11434
```

## Validation and reporting

Run these commands from the repository root:

```powershell
python -m backend.test_cube
python -m backend.dry_check
python -m backend.report openrouter
```

The test checks move permutations, inverses, known facelet output, cube identities, and BFS state counts. The dry check is safe: it only displays request bodies. The report command rebuilds `report_openrouter.md`; replace `openrouter` with `ollama` or `mock` for another mode.

## API safeguards

OpenRouter use is intentionally limited: cached answers are reused, a persistent total request cap is enforced, the Opus slot has its own cap, failures count toward the budget, and every live request is logged. Invalid model answers are counted rather than silently converted. Do not delete `cache.json` if you need to preserve the request-budget history.