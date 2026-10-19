# Personal repository validation

Validated on **19 October 2026** while preparing the personal repository. These checks validate installation artifacts and functional behavior; they do not rerun the historical research benchmarks.

## Environment

- Windows, Python 3.13.7 using the existing project virtual environment.
- Node.js 22.18.0.
- Sandbox agent provider and template narrative explainer.
- `HF_HUB_OFFLINE=1` and `TRANSFORMERS_OFFLINE=1`, with the machine's existing model caches available.
- Fresh dashboard dependency installation using the committed npm lockfile.

## Results

| Check | Result |
| --- | --- |
| Existing backend suite: `python -m pytest -q` | **49 passed**, 2 PyTorch warnings, 133.76 seconds |
| Dashboard dependency installation: `npm ci --no-audit --no-fund` | Passed; 103 packages installed |
| Production dashboard: `npm run build` | Passed; 1,591 modules transformed |
| README sandbox SDK example | Passed; created a successful recorded run |
| Built dashboard served at `/` through FastAPI TestClient | Passed; HTTP 200 with HTML content |
| `/api/health`, `/api/docs`, `/api/openapi.json` | Passed; HTTP 200 |
| `/api/llm/status`, `/api/models/status` | Passed; HTTP 200 |
| Local Markdown link destinations in current guides | Checked; targets present |
| Whitespace check: `git diff --check` | Passed |
| Credential pattern scan across reachable text history | No unreviewed matches; one known synthetic redaction-test fixture excluded |
| Personal repository history | Exactly 50 commits authored by Priyanshu Gupta; none of the 12 upstream commits is an ancestor of main |

The PyTorch warnings concern deprecated TorchScript behavior and prototype nested tensor APIs. They did not fail the existing tests. The temporary smoke-check harness explicitly closes the SDK store before removing its Windows temporary directory.

## Scope

The base Python dependencies were reused from the existing local virtual environment rather than freshly installed. Docker execution, macOS/Linux installation, optional MCP SDK compatibility, live external providers, and visual browser interactions were not exercised during this migration. No billed agent calls were requested by these checks.

The GitHub Actions workflow adds a separate Python 3.11 Ubuntu backend job and Node.js 22 dashboard build. Its remote outcome is not asserted by these local validation results. The documentation and saved benchmark artifacts distinguish functional checks from accuracy or generalization evidence.
