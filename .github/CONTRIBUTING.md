# Contributing

1. Follow the [setup guide](../docs/SETUP.md) and create a branch for a focused change.
2. Use the sandbox provider and template explainer while developing.
3. Keep generated databases, credentials, and caches out of commits.
4. Run `python -m pytest -q` for backend changes and `npm --prefix dashboard run build` for frontend changes.
5. Describe the problem, final behavior, and validation in your pull request.

Include the dataset origin, split, sample counts, and environment with new evaluation results. Keep new outputs separate from the saved metrics and model artifacts.

Use GitHub issues for reproducible bugs and feature requests. Remove private trace content from logs and screenshots. See [security guidance](SECURITY.md) for sensitive reports.
