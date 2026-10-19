# Integrations

## LangGraph

`blackbox.wrap(graph, db_path, name=..., check=...)` accepts a compiled LangGraph graph. A native checkpointer is needed for checkpoint-based fork/resume. The `check` callback returns `True`, `False`, or `None` for independent pass/fail/unknown outcomes.

Read and run [examples/expense_agent.py](../examples/expense_agent.py) for a complete example including recording, silent failure, diagnosis, fork, crash, and resume. Checkpoints retain graph state with LangGraph's reducer semantics; do not assume the adapter is a generic wrapper for every agent framework.

| Adapter | Record | Diagnose | Fork/resume | Selective reuse |
| --- | --- | --- | --- | --- |
| Built-in sandbox agent | Yes | Yes | Yes | Yes |
| LangGraph with checkpointer | Yes | Yes | Yes | No generic guarantee |
| LangGraph without checkpointer | Yes | Yes | No | No |
| Claude Code ingestion | Yes | Yes | No | No |

## MCP tools

The optional `blackbox/mcp_server.py` uses stdio and calls the running HTTP backend. Its default backend is `http://127.0.0.1:8010`; override with `BLACKBOX_URL`.

The module imports `mcp.server.mcpserver.MCPServer`, which is not installed by the base dependency list. A compatible MCP SDK exposing that interface is required. This optional integration is not validated by the base test suite; do not assume an arbitrary MCP SDK release is compatible.

With the compatible SDK installed and the backend running:

```bash
python -m blackbox.mcp_server
```

Point your MCP client's stdio configuration at that command using the repository root as its working directory. Tools include `list_failed_runs`, `diagnose_run`, `explain_run`, `test_fix`, and `failure_stats`. `test_fix` creates replay branches; the other tools inspect recorded data. Review the underlying provider and replay behavior before allowing a client to trigger experiments.

## Claude Code hooks

The sample [hook settings](../integrations/claude_code_settings.json) sends `UserPromptSubmit`, `PostToolUse`, `PostToolUseFailure`, and `Stop` events to `POST /api/ingest/claude-code`.

Merge the relevant entries into your own Claude Code configuration after reviewing them; avoid overwriting existing hooks. The sample assumes a local backend on port 8010. A command-hook alternative is provided in [claude_code_hook.py](../integrations/claude_code_hook.py), which accepts event JSON on standard input and tolerates an unavailable backend.

This adapter records and diagnoses a coding session. It does not checkpoint, fork, or resume Claude Code. Hook events can contain prompts, tool arguments, and outputs, so use a local trusted backend and inspect data before sharing reports.

## OpenTelemetry

Each run can be represented by a root agent span plus step spans with inputs, outputs, usage, and checkpoint/cache metadata. Read [TRACING.md](TRACING.md) for Phoenix and Jaeger setup.

`GET /api/runs/{run_id}/spans` provides recorded span data. Set `BLACKBOX_OTEL_CONSOLE=true` for console output or configure `OTEL_EXPORTER_OTLP_ENDPOINT` for a collector. External collectors receive trace content; use sanitized data when exploring integrations outside your machine.

The optional Compose tracing service can be started with:

```bash
docker compose --profile tracing up -d phoenix
```

Its UI is exposed at port 6006. The backend still needs the collector endpoint configured and must be restarted after environment changes.
