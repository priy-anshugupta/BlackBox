# BlackBox

### A flight recorder and debugger for AI agents

Record an agent run, investigate where it went wrong, and test a fix from a saved checkpoint. BlackBox connects execution traces, failure diagnosis, and replay in one local workspace.

**Personal project maintained by [Priyanshu Gupta](https://github.com/priy-anshugupta).** Originally developed with the NeuralForge team for the Bit and Build hackathon; this repository continues that work as a personal portfolio project. Original authorship and Git history are preserved. See [project origins](docs/PROVENANCE.md).

[Quick start](#quick-start-windows-powershell) · [Demo walkthrough](#try-the-workflow) · [Architecture](docs/ARCHITECTURE.md) · [Evaluation](docs/EVALUATION.md) · [Contributing](CONTRIBUTING.md)

## Preview

![BlackBox homepage in dark mode](dashboard/screenshots/01_home_dark.png)

<details>
<summary>Explore the investigation workspace</summary>

![Recorded agent runs](dashboard/screenshots/02_runs_overview_dark.png)
![Failure investigation and recorded evidence](dashboard/screenshots/03_run_detail_dark.png)
![Checkpoint replay and alternatives](dashboard/screenshots/05_replay_alternatives_dark.png)

</details>

Screenshots are captured from the existing project UI; appearance can vary with the selected theme and local data.

## Why I am continuing this project

Agent failures often show up several steps after the original mistake. A stale tool response can reach a calculator, a planner can lose a constraint, or a final answer can hide an earlier error. A final transcript alone makes these problems difficult to investigate.

BlackBox keeps the intermediate inputs, outputs, state, and dependencies so a developer can follow the failure backward, inspect the evidence, and compare a proposed repair against an independent check. My focus for this personal continuation is making that workflow easier to run, understand, and extend.

## Features

- **Execution recording:** per-step inputs, outputs, dependencies, token usage, and checkpoints.
- **Failure investigation:** ranked suspect steps, observable evidence, and dependency views.
- **Checkpoint replay:** test output or prompt patches while preserving the original incident.
- **Alternative fixes:** compare candidate repairs using independent acceptance checks.
- **Live recovery:** prefix checks can trigger bounded repair attempts before dependent steps run.
- **Developer integrations:** a LangGraph wrapper, Claude Code ingestion, MCP tools, and OpenTelemetry export.
- **Local dashboard:** run history, incident details, replay, comparisons, evaluation, and light/dark themes.
- **Review and export:** incident reports, regression-test export, and reviewed labels for later training.

## How it works

```text
Agent execution → Step recorder → SQLite traces + checkpoints
                                      ↓
                            Diagnosis + evidence
                                      ↓
                     Candidate fix → Fork / partial replay
                                      ↓
                       Independent checks → Comparison
```

The built-in engine supports selective reuse. External LangGraph replay uses native checkpoints and LangGraph execution semantics; its replay capabilities differ from the built-in engine. See [architecture](docs/ARCHITECTURE.md).

## Tech stack

| Layer | Technologies |
| --- | --- |
| Backend | Python, FastAPI, Uvicorn, Pydantic |
| Agent execution | LangGraph; sandbox, OpenAI, and Ollama providers |
| Diagnosis | PyTorch Transformer, LightGBM, autoencoder, sentence-transformer features |
| Storage | SQLite traces, checkpoints, and usage ledger |
| Dashboard | React 19, Vite, Tailwind CSS, Lucide icons |
| Observability | OpenTelemetry and OpenInference-style span attributes |
| Verification | pytest, FastAPI TestClient, dashboard production build |
| Packaging | Docker and Docker Compose |

## Requirements

- Python **3.11 or 3.12 recommended**; package metadata permits Python 3.10+.
- Node.js **22 LTS** and npm for the dashboard.
- Git and sufficient disk space for Python/ML dependencies and feature-model caches.
- An internet connection for initial dependency and feature-model downloads. The sandbox agent itself needs no API key.

A GPU is optional. The commands below use CPU PyTorch. Diagnosis can download pretrained feature encoders on first use; see [setup notes](docs/SETUP.md) for offline behavior.

## Quick start: Windows PowerShell

Run these commands from a terminal with Python and Node.js installed:

```powershell
git clone https://github.com/priy-anshugupta/BlackBox.git
cd BlackBox
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install --upgrade pip
.\.venv\Scripts\python.exe -m pip install torch --index-url https://download.pytorch.org/whl/cpu
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
Copy-Item .env.example .env
npm --prefix dashboard ci
npm --prefix dashboard run build
.\.venv\Scripts\python.exe -m uvicorn blackbox.api.app:app --host 127.0.0.1 --port 8010 --env-file .env
```

Open [BlackBox](http://localhost:8010). The example configuration uses `LLM_PROVIDER=sandbox`, so no paid provider is needed. Startup may take longer while feature encoders initialize. Stop the server with `Ctrl+C`.

## Quick start: macOS / Linux

```bash
git clone https://github.com/priy-anshugupta/BlackBox.git
cd BlackBox
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
cp .env.example .env
npm --prefix dashboard ci
npm --prefix dashboard run build
python -m uvicorn blackbox.api.app:app --host 127.0.0.1 --port 8010 --env-file .env
```

Open [localhost:8010](http://localhost:8010). On Linux, install a CPU-only PyTorch wheel first if you want to avoid the default accelerator dependencies. These instructions describe the supported setup path; see [validation](docs/VALIDATION.md) for what was actually checked during this migration.

## Run with Docker

With Docker Compose available, run from the repository root:

```bash
docker compose up --build
```

Open [localhost:8010](http://localhost:8010). The Compose service binds to loopback and mounts `./data` for local models, metrics, caches, and databases. `.env` is optional for Compose; copy the example if you want custom settings. The first build installs the Python and JavaScript dependencies and can take time.

Use `docker compose down` to stop the services. Local data in `./data` remains on disk. See [setup](docs/SETUP.md) for configuration details.

## Development

After installing the dependencies, use two terminals from the repository root.

Backend, on Windows:

```powershell
.\.venv\Scripts\python.exe -m uvicorn blackbox.api.app:app --reload --host 127.0.0.1 --port 8010 --env-file .env
```

Dashboard:

```bash
npm --prefix dashboard run dev
```

Open [localhost:5174](http://localhost:5174). Vite proxies `/api` requests and WebSockets to port `8010`. On macOS/Linux, use the activated virtual environment's `python` for the backend command. Rebuild the dashboard to update the version served directly by FastAPI.

## Try the workflow

1. Start the app with the sandbox provider and demo seeding enabled.
2. Choose **Open workspace** and inspect a recorded run.
3. Open a failed incident to review acceptance checks, suspect steps, and recorded evidence.
4. Use **Alternatives** to test candidate fixes from the saved checkpoint.
5. Compare the replay branch with the original and inspect which checks passed or failed.
6. Use **Live run** to record a new task; the built-in families cover finance, SQL, document QA, and math.

A successful replay supports a proposed explanation. It does not prove that the suggested step is the only possible cause. For an external graph example, run `python examples/expense_agent.py` in your virtual environment.

## Use the Python SDK

From the repository root with dependencies installed:

```python
import blackbox

agent = blackbox.wrap(
    blackbox.SandboxAgent(task_family="finance"),
    db_path="data/my-agent.db",
    llm="sandbox",
)
run = agent.invoke({
    "prompt": "Convert INR 50,000 to USD and compute EMI for 12 months at 9%."
})
print(run["run_id"])
```

`blackbox.wrap` also accepts a compiled LangGraph graph. Native checkpointing enables fork/resume; provide an independent `check(final_state)` callback to label external outcomes. See [integrations](docs/INTEGRATIONS.md) and [the expense agent](examples/expense_agent.py).

## Provider configuration

| Provider | Configuration | Intended use |
| --- | --- | --- |
| Sandbox | `LLM_PROVIDER=sandbox` | Local demo and deterministic agent behavior without API credentials |
| Ollama | `LLM_PROVIDER=ollama`, `OLLAMA_MODEL`, `OLLAMA_BASE_URL` | An optional locally hosted language model |
| OpenAI | `LLM_PROVIDER=openai`, `OPENAI_API_KEY`, `OPENAI_MODEL` | Optional billed agent calls |

Copy `.env.example` to `.env` and configure only the provider you need. Provider availability and active models are visible through `GET /api/llm/status`. Diagnosis uses local model artifacts; the optional narrative explainer has a separate provider setting.

For billed experiments, review model availability, prices, and the usage budget before running them. The usage ledger is a local accounting control; it is not a provider-side billing limit. Never commit `.env` or credentials.
