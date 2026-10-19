# Contributing

BlackBox is maintained by [Priyanshu Gupta](https://github.com/priy-anshugupta) as a personal continuation of a team hackathon project. Read [provenance](docs/PROVENANCE.md) before reusing or redistributing project code; no new project-wide license is granted here.

## Development workflow

1. Follow the [README setup](README.md) and create a branch for a focused change.
2. Use the sandbox provider and template explainer while developing.
3. Keep generated databases, credentials, caches, and local configuration out of commits.
4. Run `python -m pytest -q` for backend changes and `npm --prefix dashboard run build` for frontend changes.
5. Describe the problem, final behavior, validation performed, and any limits in your pull request.

Use real changes and present-day commits. Preserve original authorship and avoid rewriting shared history. New evaluation data should include its origin, sample counts, split, and environment; keep it separate from the archived final results.

## Reports and requests

Use GitHub issues for reproducible bugs or scoped feature requests. Include setup details and sanitized logs. Screenshots should avoid personal prompts, API keys, and private trace data. For sensitive reports, follow [SECURITY.md](SECURITY.md).
