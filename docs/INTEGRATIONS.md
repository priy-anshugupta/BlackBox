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
