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
