# Architecture

## Recording and persistence

The React dashboard calls the FastAPI application in `blackbox/api/app.py`. The API delegates built-in task execution to `Engine`; `Store` persists runs and trace state in SQLite. Durable events support run streams and late subscribers.

The built-in agent in `blackbox/agent/` uses planner, router, tool, reasoning, and final-answer stages. Each step records its inputs, outputs, parent step IDs, state/checkpoint information, and usage metadata. Content caching and explicit dependencies let the engine identify reusable work during an experiment.

The recorded trace is the inspection surface. Secret redaction is implemented in storage, but callers should still avoid recording sensitive inputs. Redaction is not a complete data-loss prevention system.
