# Architecture

## Recording and persistence

The React dashboard calls the FastAPI application in `blackbox/api/app.py`. The API delegates built-in task execution to `Engine`; `Store` persists runs and trace state in SQLite. Durable events support run streams and late subscribers.

The built-in agent in `blackbox/agent/` uses planner, router, tool, reasoning, and final-answer stages. Each step records its inputs, outputs, parent step IDs, state/checkpoint information, and usage metadata. Content caching and explicit dependencies let the engine identify reusable work during an experiment.

The recorded trace is the inspection surface. Secret redaction is implemented in storage, but callers should still avoid recording sensitive inputs. Redaction is not a complete data-loss prevention system.

## Diagnosis and evidence

`features/` extracts semantic and numeric step features. The trained diagnosis ensemble combines a Transformer, LightGBM, and autoencoder; the saved ensemble uses weights 0.6, 0.3, and 0.1. Model availability and fallback status are part of interpreting the output.

`diagnosis.py` produces ranked suspects and evidence. `explain/` builds the supporting narrative, feature factors, trace comparisons, and data-flow information. The default narrative is template-based; optional language-model narration is separate from the diagnosis model.

`verifier.py` evaluates independent acceptance criteria such as source freshness, agreement, argument consistency, and answers. A model's explanation does not determine whether a repair passes. Unknown checks should remain unknown rather than becoming implicit successes.
