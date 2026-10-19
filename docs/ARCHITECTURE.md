# Architecture

## Recording and persistence

The React dashboard calls the FastAPI application in `blackbox/api/app.py`. The API delegates built-in task execution to `Engine`; `Store` persists runs and trace state in SQLite. Durable events support run streams and late subscribers.

The built-in agent in `blackbox/agent/` uses planner, router, tool, reasoning, and final-answer stages. Each step records its inputs, outputs, parent step IDs, state/checkpoint information, and usage metadata. Content caching and explicit dependencies let the engine identify reusable work during an experiment.

The recorded trace is the inspection surface. Secret redaction is implemented in storage, but callers should still avoid recording sensitive inputs. Redaction is not a complete data-loss prevention system.

## Diagnosis and evidence

`features/` extracts semantic and numeric step features. The trained diagnosis ensemble combines a Transformer, LightGBM, and autoencoder; the saved ensemble uses weights 0.6, 0.3, and 0.1. Model availability and fallback status are part of interpreting the output.

`diagnosis.py` produces ranked suspects and evidence. `explain/` builds the supporting narrative, feature factors, trace comparisons, and data-flow information. The default narrative is template-based; optional language-model narration is separate from the diagnosis model.

`verifier.py` evaluates independent acceptance criteria such as source freshness, agreement, argument consistency, and answers. A model's explanation does not determine whether a repair passes. Unknown checks should remain unknown rather than becoming implicit successes.

## Replay and alternatives

Replay creates a new linked run, preserving the original incident. The built-in engine starts from a saved checkpoint, applies an intervention, and re-executes affected work. Recorded and fresh replay modes answer different questions about cached observations and new tool execution.

`strategies.py` describes candidate repairs and their preconditions. Experiments can test several candidates from one incident. Results include independent check outcomes, cost/usage, and the relationship between the original and replay branches.

External LangGraph graphs use native graph checkpoints. Their reducers and scheduling rules remain controlled by LangGraph. Graphs without a checkpointer can be recorded but cannot fork or resume. The SDK capability map explicitly does not promise the built-in engine's selective reuse for arbitrary external graphs.

## Live recovery and review

`live.py` evaluates prefixes after each step, before dependent steps execute. Recovery attempts are bounded; unsuitable incidents can be escalated instead of continually retried. The built-in checks were written with the known fault catalogue in mind, which limits claims about unseen failures.

`memory.py` finds strategies that previously passed in similar tested incidents. `review_queue.py` records human confirmation/correction of suspects, exports labels, and supports training into a separate model directory before promotion.

`report.py` generates incident reports and `regression.py` exports failures as pytest regression checks. These exports help turn an investigation into a repeatable engineering artifact; exported checks still need review for environment and data assumptions.
