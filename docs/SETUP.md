# Setup and local operation

Start with the platform-specific commands in [the README](../README.md). Run all commands from the repository root unless stated otherwise.

## Runtime and hardware

Python 3.11/3.12 is recommended for dependency compatibility. The project declares Python 3.10+; the migration also verifies against the existing Windows Python 3.13 environment. Node.js 22 matches the dashboard Docker build. No GPU is required for the sandbox or CPU diagnosis.

The pretrained semantic/NLI feature encoders can be downloaded from Hugging Face during the first initialization. Internet-free agent execution does not guarantee an internet-free first installation. Existing model caches can be reused; an unavailable encoder may cause the feature extractor to fall back rather than provide the trained semantic representation.

For GPU inference, install the PyTorch build appropriate to your driver before the other requirements. CPU installation is the simplest starting point. Avoid assuming that the recorded GPU latency applies to your computer.

## Configuration

Copy `.env.example` to `.env`. Environment variables already exported in the shell take precedence over values loaded by the application. Restart the backend after changing settings because configuration is cached.

| Variable | Purpose |
| --- | --- |
| `LLM_PROVIDER` | `sandbox`, `ollama`, or `openai` |
| `BLACKBOX_DB` | Trace database path, default `data/traces.db` |
| `BLACKBOX_MODEL_DIR` | Diagnosis model directory |
| `BLACKBOX_METRICS` | Saved evaluation JSON exposed by the API |
| `BLACKBOX_SEED_DEMO` | Seed examples at startup when the database is empty |
| `BLACKBOX_USAGE_DB` | Local usage ledger database |
| `EXPLAINER_NARRATIVE` | Template narrative or optional LLM narrative |
| `BLACKBOX_CORS_ORIGINS` | Allowed dashboard development origins |

For a separate demo database in PowerShell:

```powershell
$env:BLACKBOX_DB = "data/personal-demo.db"
```

Use a new database path instead of deleting your investigations. Databases, caches, generated datasets, and local secrets are intentionally ignored by Git. The committed `data/models/` and metric JSON files are part of the project evidence; back them up before experimenting.

Compose mounts `./data` into the container and uses `/app/data` paths. The optional `phoenix` tracing service only starts when its profile is selected.

## Dashboard modes

For one server, build with `npm --prefix dashboard run build`, then open port 8010. FastAPI serves `dashboard/dist`. For development, run Vite on port 5174 and the API on port 8010. Vite has a strict port setting and proxies `/api` traffic, including WebSockets.

## Troubleshooting

| Symptom | Check |
| --- | --- |
| Dashboard unavailable but API works | Build `dashboard/dist`, then restart the backend |
| Vite refuses to start | Port 5174 is occupied; stop the other process or change the Vite configuration |
| Agent provider unavailable | Inspect `/api/llm/status`; confirm the provider service/key and restart |
| Slow first startup | Feature encoders may be downloading or loading; inspect server logs |
| Diagnosis uses a fallback | Check `/api/models/status`, artifact paths, and available feature encoders |
| Old incident still appears failed after a repair | Repairs create linked branches; the original remains unchanged |
| No demo runs | Demo seeding only populates an empty database and must be enabled |
| MCP import fails | The MCP module is optional and its SDK is not declared by the base requirements; see [integrations](INTEGRATIONS.md) |

For offline validation, export `HF_HUB_OFFLINE=1` and `TRANSFORMERS_OFFLINE=1`. This uses locally available encoders or fallback behavior and must not be treated as reproduction of the published trained-model metrics.
