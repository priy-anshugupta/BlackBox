# Security and data handling

## Intended environment

The current API is a local development application without built-in authentication or authorization. Bind it to `127.0.0.1` and use trusted local clients. CORS is not authentication. Add appropriate access controls and review data handling before exposing the service to other users.

## Data to keep private

Traces can include prompts, tool arguments, outputs, and state. Storage has redaction support, but it cannot guarantee removal of every secret or personal detail. Inspect exported reports and screenshots before sharing. Keep `.env`, credentials, databases, caches, and webhook URLs out of Git.

Use only trusted model artifacts: deserialization of model files can be a security boundary. Optional collectors and alert webhooks send data outside the process; configure destinations intentionally.

## Reporting a sensitive issue

Do not put credentials or exploitable details in a public issue. Use GitHub private vulnerability reporting if enabled for this repository, or contact the maintainer through a private channel you already know. No dedicated security email address or guaranteed response time is advertised.

For a non-sensitive bug, use the normal issue template with sanitized reproduction steps.
