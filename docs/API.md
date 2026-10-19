# HTTP API

Start the backend on `http://127.0.0.1:8010`. The examples use the `/api` prefix, which is also used by the dashboard proxy.

## Discovery and health

- Interactive schema: [http://localhost:8010/api/docs](http://localhost:8010/api/docs)
- OpenAPI JSON: `GET /api/openapi.json`
- Health: `GET /api/health`
- Provider configuration/status: `GET /api/llm/status`
- Diagnosis artifact status: `GET /api/models/status`

PowerShell:

```powershell
Invoke-RestMethod http://127.0.0.1:8010/api/health
```

The runtime schema is the authoritative reference for request fields. The application currently has no authentication layer; use it on a trusted local interface.

## Record a run

```powershell
$body = @{
    prompt = "Convert INR 50,000 to USD and compute EMI for 12 months at 9%."
    task_family = "finance"
    provider = "sandbox"
} | ConvertTo-Json
$created = Invoke-RestMethod -Method Post -Uri http://127.0.0.1:8010/api/runs -ContentType application/json -Body $body
$created.run_id
```

Creation returns HTTP 202 with a run ID. Fetch the run with `GET /api/runs/{run_id}` or connect to `ws://127.0.0.1:8010/api/runs/{run_id}/stream`. Streams contain durable step and terminal events; an `after` event cursor lets a client resume. Run retrieval immediately after creation may show queued/running state.

Supported built-in families are `finance`, `sql`, `doc_qa`, and `math`. Unknown fields and invalid inputs are rejected by the API request models.

## Replay and experiments

`POST /api/runs/{run_id}/replay` accepts a positive `from_step`, a patch, replay count `k` from 1 to 10, and a mode (`recorded` or `fresh`). Use the actual step and output schema from the recorded run when constructing a patch.

For example, this is the request shape for a manually supplied output intervention:

```json
{
  "from_step": 3,
  "patch": {"output": {"value": 52.47}},
  "k": 3,
  "mode": "recorded"
}
```

The value above is illustrative, not a universal repair. Inspect `GET /api/runs/{run_id}/suggest-fix?step=3` or the alternatives endpoint for incident-specific candidates. `POST /api/runs/{run_id}/experiments` runs strategies from a checkpoint and can explicitly control billed-provider permission with `allow_billed`.

Experiments create linked runs. Compare two run IDs with `GET /api/compare?a={before_id}&b={after_id}`. Keep intervention type, check outcomes, and reused work in the interpretation of any repair result.

## Endpoint map

| Method and route | Purpose |
| --- | --- |
| `GET /api/runs` | Filter/list recorded runs |
| `GET /api/runs/{id}` | Trace, status, diagnosis, and evidence |
| `POST /api/runs/{id}/inject` | Create a fault-injected branch |
| `GET /api/runs/{id}/alternatives` | Candidate strategies |
| `GET /api/runs/{id}/experiments` | Existing strategy experiment results |
| `GET /api/runs/{id}/explain` | Narrative explanation |
| `GET /api/runs/{id}/report` | Incident report |
| `GET /api/runs/{id}/regression-test` | Regression-test export |
| `POST /api/runs/{id}/review` | Confirm, correct, or mark uncertain |
| `GET /api/runs/{id}/memory` | Relevant tested strategies |
| `GET /api/stats` | Run and failure statistics |
| `GET /api/eval` | Saved evaluation metrics |
| `GET /api/faults` | Built-in fault catalogue |

Use `/api/docs` for complete parameter and response definitions. A successful HTTP response alone does not imply the agent task passed; inspect the run and its independent checks.
