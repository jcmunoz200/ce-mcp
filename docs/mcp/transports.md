# Transports

## Summary

| Transport | Status | Default | Use case |
| --- | --- | --- | --- |
| stdio | Standard, implemented by `rmcp` | Yes | One local agent launches the server |
| Streamable HTTP | Standard, implemented by `rmcp` | Optional | Multi-client, embedded app, debugging |
| In-process worker | `rmcp` feature | Optional | Future desktop shell |
| Unix domain socket | Custom | Deferred | Lower overhead than loopback TCP |
| WebSocket | Not standard | No | Not supported |
| HTTP+SSE (legacy) | Deprecated | No | Not supported |

## stdio

The client launches `ce-mcp` as a subprocess and exchanges newline-delimited
JSON-RPC over stdin/stdout.

```json
{
  "mcpServers": {
    "ce-mcp": {
      "command": "ce-mcp",
      "args": ["--profile", "core"]
    }
  }
}
```

Properties:

- no listening port and no network surface;
- the client controls the process lifecycle;
- logs must go to stderr, never stdout;
- one client per process;
- the fastest practical option for a local agent.

The bridge still uses a localhost TCP socket to reach Cheat Engine. That is a
separate connection, not the MCP transport.

## Streamable HTTP

The server exposes a single MCP endpoint, for example
`http://127.0.0.1:7337/mcp`. Each message is an HTTP POST. The response is a JSON
object or a request-scoped SSE stream.

```json
{
  "mcpServers": {
    "ce-mcp": {
      "url": "http://127.0.0.1:7337/mcp"
    }
  }
}
```

Properties:

- multiple clients can share one server;
- useful for the embedded desktop mode and for inspecting traffic;
- must bind to `127.0.0.1` by default;
- requires a token or another local authentication mechanism before it is
  exposed beyond a single-user desktop;
- supports request-scoped SSE for progress, but the design must not require SSE
  for correctness;
- no session ID; state is in the backend handles.

### Security defaults

- Listen address: `127.0.0.1` only.
- Port: chosen at startup; written to the state file.
- Token: required for non-stdio transports.
- Origin checks: enabled for browser-originated requests.
- Public bind: requires an explicit `--allow-remote` flag and a documented
  warning.
- No CORS wildcard.

## In-process worker

`rmcp` supports an in-process worker transport. It is a good fit for a future
desktop shell that hosts the server and the UI in one process:

- no serialization over a socket;
- direct Rust calls;
- still uses the same tool registry and backend contract.

This transport is not part of the first release.

## Unix domain socket

A Unix domain socket would avoid the loopback TCP stack and could use the stdio
framing. It is deferred because MCP clients do not universally support custom
transports and because TCP is already local and fast enough.

If added, it must be opt-in and use the same authentication and framing rules.

## Why not WebSocket?

WebSocket is not a standard MCP transport in `2026-07-28`. Adding it would
require a custom client. Streamable HTTP already provides request-scoped SSE for
streaming and works with standard clients.

## Performance

The transport is rarely the bottleneck:

- a memory read is dominated by `task_for_pid`/Mach VM access and Lua
  serialization;
- a first scan is dominated by the scan engine;
- a debugger wait is dominated by the target.

stdio avoids HTTP parsing and connection management, so it is the default. On
loopback, Streamable HTTP adds roughly a sub-millisecond round trip plus HTTP and
SSE framing; that is negligible compared with a scan.

Do not choose HTTP for speed. Choose it for multi-client or embedded use.

## Backward compatibility

On stdio, a client that supports both modern and legacy servers should send
`server/discover` first. If the server answers, the client uses the modern
stateless flow. If it does not, the client may fall back to the `initialize`
handshake.

`ce-mcp` will advertise both `2026-07-28` and `2025-11-25` during the
compatibility period. The F0 spike must test both paths with real clients.

## References

- [MCP transports](https://modelcontextprotocol.io/specification/2026-07-28/basic/transports)
- [stdio](https://modelcontextprotocol.io/specification/2026-07-28/basic/transports/stdio)
- [Streamable HTTP](https://modelcontextprotocol.io/specification/2026-07-28/basic/transports/streamable-http)
- [ADR-0003](../decisions/ADR-0003-transports.md)
