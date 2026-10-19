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
