# MCP research

## Target revision

The design targets **MCP `2026-07-28`**, the current stable revision at the time
of writing.

Key changes from `2025-11-25`:

- **Stateless protocol.** The `initialize` handshake, `notifications/initialized`
  and `Mcp-Session-Id` are removed.
- **Per-request metadata.** Protocol version and client capabilities travel in
  `_meta` on every request; clients should also identify themselves there.
- **`server/discover`.** Servers advertise supported versions, capabilities and
  identity. Clients may use it for version selection or as a compatibility
  probe.
- **`subscriptions/listen`.** Replaces the standalone HTTP GET stream and
  `resources/subscribe`/`unsubscribe`. Clients opt in to notification
  categories.
- **Tasks extension.** Long-running work is a separate extension with
  `tasks/get`, `tasks/update` and `tasks/cancel`. The old blocking
  `tasks/result` pattern is gone.
- **Caching hints.** List and resource results carry `ttlMs` and `cacheScope`.
- **Multi-round-trip requests.** Servers return `InputRequiredResult` with
  embedded requests instead of initiating separate server-to-client requests.
- **`resultType`.** Every result is `complete`, `input_required` or, when the
  tasks extension is active, `task`.
- **No SSE resumability.** A broken response stream loses the in-flight
  request; clients re-issue it with a new request ID.

Sources:

- Specification: <https://modelcontextprotocol.io/specification/2026-07-28>
- Changelog: <https://modelcontextprotocol.io/specification/2026-07-28/changelog>
- Transports: <https://modelcontextprotocol.io/specification/2026-07-28/basic/transports>

## Transports

The standard bindings are:

1. **stdio** — newline-delimited JSON-RPC over the standard streams of a
   client-launched subprocess.
2. **Streamable HTTP** — each message is an HTTP POST to a single endpoint;
   replies are a JSON object or a request-scoped SSE stream.

The old HTTP+SSE transport is deprecated. WebSocket is **not** a standard MCP
transport. The specification allows custom transports, and recommends reusing
the stdio framing for reliable bidirectional byte streams such as Unix domain
sockets.

| Transport | Local latency | Clients | Notes |
| --- | --- | --- | --- |
| stdio | Lowest | One per process | No HTTP overhead; best for a local agent |
| Streamable HTTP | ~sub-millisecond on loopback, plus HTTP/SSE overhead | Many | Best for the embedded app and multi-client |
| In-process worker | No network serialization | One | Best for UI ↔ server inside one process |
| HTTP over Unix socket | Lower than TCP | Many local | Optional; not universally supported by clients |

The transport is not the bottleneck for Cheat Engine operations. Scans and
debugger waits dominate. The design therefore uses **stdio by default** and
**Streamable HTTP on localhost** when a shared or multi-client server is needed.
It does not invent a new external transport.

## Rust SDK

The planned implementation uses **`rmcp` 3.6**, the official Rust SDK.

Relevant capabilities observed in its documentation:

- MCP `2026-07-28` support with compatibility for `2025-11-25` and earlier;
- `server/discover` and discovery-based client lifecycle;
- stateless Streamable HTTP with JSON responses or request-scoped SSE;
- tasks extension support;
- subscriptions and `subscriptions/listen`;
- response caching with `ttlMs`/`cacheScope`;
- multi-round-trip requests;
- stdio, child process, Streamable HTTP and in-process worker transports.

Features planned for `ce-mcp`:

```toml
rmcp = { version = "3.6", features = [
  "server",
  "transport-io",
  "transport-streamable-http-server",
  "transport-worker",
  "macros"
] }
```

## Design consequences

### Stateless server, stateful engine

MCP has no sessions, so the server cannot keep per-connection state. The backend
keeps state and returns explicit handles:

- `process_open` returns a `process_handle`;
- `scan_start` returns a `scan_handle` and possibly a task ID;
- `debug_attach` returns a `debug_handle`;
- `table_new`/`table_load` return a `table_handle`.

Handles are opaque, have a TTL and can be closed explicitly.

### Tasks for long scans

A first scan or pointer scan can outlive a normal request. The server can
materialize a task and return a task handle. The client polls `tasks/get` and
can cancel with `tasks/cancel`. This is preferable to holding an HTTP request
open for minutes.

### Caching tool lists

`tools/list` is deterministic and carries `ttlMs`/`cacheScope`. The `lua` and
`all` profiles list hundreds of tools; caching is important for client startup
and prompt cache stability.

### No sampling, roots or logging dependency

The `2026-07-28` revision deprecates roots, sampling and logging. The design
does not depend on them. Logs go to stderr in stdio mode and to a file or
OpenTelemetry in server mode.

## Open questions

- How much of the `2025-11-25` compatibility path F0 must prove. The current
  target is the stdio path; HTTP compatibility can wait for F1. `rmcp` provides
  the mechanism, but the bridge spike must test it with a real legacy client.
- Whether the embedded desktop mode needs HTTP at all in F1, or whether stdio
  is sufficient until multi-client is required.
- How to expose progress without forcing SSE for every client.

These questions do not block D0.
