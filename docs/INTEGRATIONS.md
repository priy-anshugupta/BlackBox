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
