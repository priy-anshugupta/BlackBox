# Development guide

## Architecture

The React dashboard calls the FastAPI application in `blackbox/api/app.py`. The built-in `Engine` executes LangGraph tasks, while `Store` persists runs and checkpoints in SQLite. Durable events provide live streams and allow late subscribers to recover earlier events.

```text
Agent → Recorder → Traces and checkpoints → Diagnosis and evidence
                                                    ↓
                       Comparison ← Checks ← Replay with a patch
```

Diagnosis combines semantic and numeric step features with a Transformer, LightGBM, and autoencoder ensemble. Explanations describe observable traces. Independent acceptance checks determine whether a replay passes.

Replay creates a linked run and preserves the original incident. The built-in engine can reuse unaffected work; external LangGraph replay follows native checkpoints and graph scheduling. Live recovery checks prefixes before dependent steps execute and bounds repair attempts.

## Development servers

After following [setup](SETUP.md), start the backend in your active virtual environment:

```bash
python -m uvicorn blackbox.api.app:app --reload --host 127.0.0.1 --port 8010 --env-file .env
```

In another terminal:

```bash
npm --prefix dashboard run dev
```

Open [localhost:5174](http://localhost:5174). Vite proxies `/api` requests and WebSockets to port 8010. Rebuild the dashboard to update the static version served by FastAPI.

## Python SDK

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
agent.engine.store.close()
```

`blackbox.wrap` also accepts compiled LangGraph graphs. Supply an independent `check(final_state)` callback for pass/fail/unknown outcomes. Native checkpointing is required for fork/resume. See [the expense agent](../examples/expense_agent.py) for a complete example.

## HTTP API

The runtime schema at [localhost:8010/api/docs](http://localhost:8010/api/docs) is the reference for complete request and response definitions.

| Endpoint | Purpose |
| --- | --- |
| `GET /api/health` | Backend health |
| `GET /api/llm/status`, `GET /api/models/status` | Provider and diagnosis status |
| `POST /api/runs` | Record a task; returns HTTP 202 and a run ID |
| `GET /api/runs/{id}` | Trace, outcome, diagnosis, and evidence |
| `WS /api/runs/{id}/stream` | Durable run events; supports an `after` cursor |
| `GET /api/runs/{id}/alternatives` | Candidate fixes |
| `POST /api/runs/{id}/replay` | Checkpoint replay with a patch |
| `POST /api/runs/{id}/experiments` | Compare strategies |
| `GET /api/compare?a={before}&b={after}` | Compare branches |
| `GET /api/runs/{id}/report` | Incident report |
| `POST /api/runs/{id}/review` | Human confirmation or correction |
| `GET /api/eval` | Saved evaluation metrics |

Built-in families are `finance`, `sql`, `doc_qa`, and `math`. Construct replay patches from the actual step/output schema; check the independent outcomes rather than treating HTTP success as task success.

## Integrations

- **LangGraph:** recording is supported with or without a checkpointer; fork/resume requires one. Arbitrary external graphs do not have the built-in engine's selective-reuse guarantee.
- **Claude Code:** review and merge [the hook settings](../integrations/claude_code_settings.json) into your configuration. Events go to `/api/ingest/claude-code`. This records and diagnoses sessions; it does not fork/resume them.
- **MCP:** `python -m blackbox.mcp_server` exposes diagnosis tools through stdio and requires a running backend. It also requires a compatible SDK exposing `mcp.server.mcpserver.MCPServer`, which is not included in the base requirements. Set `BLACKBOX_URL` to change the backend address. `test_fix` creates replay branches.
- **OpenTelemetry:** `GET /api/runs/{id}/spans` exports recorded spans. Set `BLACKBOX_OTEL_CONSOLE=true` for console output or configure `OTEL_EXPORTER_OTLP_ENDPOINT` for a collector. The optional Phoenix service starts with `docker compose --profile tracing up -d phoenix` and exposes port 6006.

## Verification

```bash
python -m pytest -q
npm --prefix dashboard run build
```

Use the sandbox provider and template explainer for development. Tests cover recording, diagnosis, replay, verification, API behavior, and graph handling. Missing feature encoders can change diagnosis predictions; use the intended pretrained encoders when checking model-dependent behavior.

Keep generated data and new experiment outputs separate from the committed models and metrics. Review exported reports, tracing destinations, and hook payloads before sharing sensitive data.
