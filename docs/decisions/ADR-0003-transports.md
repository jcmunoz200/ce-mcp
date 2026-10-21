# ADR-0003: Use stdio by default and Streamable HTTP on localhost

- **Status:** Accepted
- **Date:** 2026-10-21
- **Supersedes:** none
- **Superseded by:** none

## Context

MCP `2026-07-28` defines two standard transports:

- **stdio**, where the client launches the server as a subprocess and exchanges
  newline-delimited JSON-RPC over stdin/stdout;
- **Streamable HTTP**, where each message is an HTTP POST to one endpoint and
  the response is either a JSON object or a request-scoped SSE stream.

The old HTTP+SSE transport is deprecated. WebSocket is not a standard MCP
transport. The specification allows custom transports and recommends reusing
the stdio framing for reliable bidirectional streams such as Unix domain
sockets.

The project has two likely deployment modes:

1. a single local agent that launches the server;
2. an embedded or desktop mode where several clients share one server.

## Decision

- **stdio is the default.** It has the lowest overhead, no network surface and
  the simplest lifecycle.
- **Streamable HTTP is optional and bound to localhost.** It exists for
  multi-client and embedded use.
- **No public bind by default.** Binding to a non-loopback address requires an
  explicit flag and a token.
- **No WebSocket or custom external transport.** A Unix domain socket may be
  added later using the stdio framing, but it is not part of the initial
  contract.
- **The in-process worker transport** may be used by the future desktop shell;
  it does not change the wire contract.

## Consequences

### Positive

- The common case has no HTTP server, port or CORS surface.
- stdio is the fastest practical local path and works with most MCP clients.
- HTTP covers multi-client use without inventing a transport.
- The same tool registry and backend serve both transports.

### Negative

- Two transport code paths must be tested.
- HTTP mode needs authentication, origin checks and lifecycle management.
- stdio mode logs to stderr and must never write protocol data there.
- Some clients only support one transport, so documentation must be explicit.

### Neutral

- Transport choice does not affect tool schemas.
- The transport is not the performance bottleneck; scans and debugger waits
  dominate.

## Alternatives considered

1. **HTTP only.** Rejected: unnecessary network surface and overhead for the
   common single-agent case.
2. **stdio only.** Rejected: does not support the embedded multi-client mode.
3. **WebSocket.** Rejected: not a standard MCP transport in this revision.
4. **gRPC or a custom binary protocol.** Rejected: clients would need custom
   support; MCP already defines the contract.
5. **Unix domain socket only.** Deferred: lower overhead than TCP, but not
   universally supported by MCP clients.

## References

- [Transports](../mcp/transports.md)
- [MCP transports specification](https://modelcontextprotocol.io/specification/2026-07-28/basic/transports)
- [ADR-0002](ADR-0002-stateless-handles.md)
