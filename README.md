# BlackBox

A flight recorder and debugger for AI agents. Record executions, investigate failures, and test fixes from saved checkpoints in a local dashboard.

![BlackBox dashboard](dashboard/screenshots/01_home_dark.png)

## Features

- **Record** step inputs, outputs, state, dependencies, and token usage.
- **Investigate** failures with ranked suspects and supporting evidence.
- **Replay** from checkpoints and compare repairs with the original run.
- **Verify** candidate fixes with independent acceptance checks.
- **Integrate** with LangGraph, Claude Code hooks, and OpenTelemetry.

**Stack:** Python · FastAPI · LangGraph · PyTorch · LightGBM · SQLite · React · Vite · Tailwind CSS

## Quick start

Use Python 3.11/3.12 and Node.js 22. No API key or GPU is needed for the sandbox demo. Initial setup downloads dependencies and feature models.

```bash
git clone https://github.com/priy-anshugupta/BlackBox.git
cd BlackBox
python -m venv .venv
```

Activate the environment:

```powershell
# Windows PowerShell
.\.venv\Scripts\Activate.ps1
```

```bash
# macOS / Linux
source .venv/bin/activate
```

Install dependencies and build the dashboard:

```bash
python -m pip install -r requirements.txt
npm --prefix dashboard ci
npm --prefix dashboard run build
```

Copy `.env.example` to `.env` (`Copy-Item .env.example .env` on PowerShell or `cp .env.example .env` on macOS/Linux), then start the app:

```bash
python -m uvicorn blackbox.api.app:app --host 127.0.0.1 --port 8010 --env-file .env
```

Open [localhost:8010](http://localhost:8010). Select **Open workspace** to inspect a run, review a failed step, and test alternatives. The default provider is the local sandbox; OpenAI and Ollama are optional.

<details>
<summary>Docker</summary>

```bash
docker compose up --build
```

Open the same address. Local models, metrics, and databases persist in `data/`.

</details>

## Development

```bash
python -m pytest -q
npm --prefix dashboard run dev
```

Vite runs on port `5174` and proxies API requests to the backend on `8010`. API documentation is available at [localhost:8010/api/docs](http://localhost:8010/api/docs).

## Project structure

| Directory | Purpose |
| --- | --- |
| `blackbox/` | Backend, recording, diagnosis, replay, and model code |
| `dashboard/` | React interface and screenshots |
| `data/` | Saved metrics and trained model artifacts |
| `docs/` | Setup, development, and evaluation guides |
| `examples/`, `integrations/` | SDK examples and external adapters |
| `scripts/`, `tests/` | Development helpers and automated checks |

## Documentation

- [Setup and troubleshooting](docs/SETUP.md)
- [Architecture, SDK, and API](docs/DEVELOPMENT.md)
- [Evaluation and limitations](docs/EVALUATION.md)

BlackBox is a research prototype intended for trusted local use. Diagnosis quality varies across workflows; saved benchmark results and their limitations are documented in the evaluation guide.
