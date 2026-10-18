# x4-mcp-prod

**Production-grade MCP gateway**: registry, authentication, permissions, schema validation, rate limits, and audit logging.

Ready for multi-agent fleets. Complements the lighter `x4-mcp` / `x4-mcpgen` tools.

## Features

- Explicit tool allow-lists (no `*`)
- Per-client rate limiting
- Schema validation on every call
- Audit trail compatible with x4-evidence
- stdio + HTTP transports

## Quick Start

```bash
pip install -e ".[dev]"
x4-mcp-prod serve --config config/example.yml
```

## License

Apache-2.0

## APEX Architecture Map

> Status: architecture documentation; implementation and deployment claims require independent verification.

```mermaid
flowchart TB
  Client([MCP client]) --> Transport[Transport adapter]
  Transport --> Protocol[Protocol handler]
  Protocol --> Validate[Validate message and tool schema]
  Validate --> Identity[Authenticate caller]
  Identity --> Rate{Rate / quota available?}
  Rate -->|no| Throttle[Return bounded throttle error]
  Rate -->|yes| Authorize{Tool and scope allowed?}
  Authorize -->|no| Deny[Return permission error]
  Authorize -->|yes| Execute[Invoke backend adapter]
  Execute --> Normalize[Normalize result and errors]
  Normalize --> Audit[Emit structured audit event]
  Normalize --> Metrics[Record latency / counts]
  Metrics --> Exporter[Metrics exporter]
  Audit --> Response[Protocol response]
  Throttle --> Response
  Deny --> Audit
  Response --> Client
```
**Evidence note:** The diagram describes production-oriented target controls. Verify transport, authentication, quotas, metrics, and backend adapters before describing the server as production-ready.

