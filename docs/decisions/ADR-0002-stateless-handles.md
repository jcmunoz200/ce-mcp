# ADR-0002: Use stateless MCP with explicit handles

- **Status:** Accepted
- **Date:** 2026-10-21
- **Supersedes:** none
- **Superseded by:** none

## Context

MCP `2026-07-28` removes the initialization handshake and sessions. Every
request is self-describing: protocol version, client capabilities and client
information travel in `_meta`. There is no `Mcp-Session-Id` and no per-connection
server state.

Cheat Engine is stateful. It has an open process, a current scan, an address
list, a debugger session, symbols, hotkeys and global settings. Pretending
otherwise would make the tools unreliable, especially with multiple clients.

## Decision

The MCP layer is stateless. The backend owns state and exposes it through opaque
handles.

| Handle | Created by | State |
| --- | --- | --- |
| `process_handle` | `process_open` | Open process, including start time to detect PID reuse |
| `scan_handle` | `scan_start` | Scan type, parameters, results and progress |
| `debug_handle` | `debug_attach` | Debugger session and breakpoints |
| `table_handle` | `table_new` / `table_load` | Address-list/table context |

Rules:

- Handles are opaque strings. Clients must not parse them.
- Every handle has a TTL and a creation timestamp.
- `ce_handles` lists live handles and their state.
- Tools that need state require the corresponding handle explicitly.
- Closing a handle releases backend resources. Process exit invalidates the
  handle and returns `CE_PROCESS_CLOSED`.
- The server may keep a default handle for convenience, but it must be visible
  in `ce_status` and must not be required.
- Long operations return a task ID in addition to the handle.

## Consequences

### Positive

- Compatible with the stateless MCP revision.
- Multiple clients can share one bridge without invisible session state.
- State is observable and can be cleaned up.
- A server restart does not silently reuse a stale process.

### Negative

- Clients must pass handles and manage their lifetime.
- A careless client can leak handles until the TTL expires.
- The backend needs a handle registry, TTL sweeper and invalidation logic.
- Tool signatures are slightly more verbose.

### Neutral

- Handles are not security boundaries. The token and localhost binding are.
- The default handle is a convenience, not a protocol feature.

## Alternatives considered

1. **Per-connection sessions.** Rejected: MCP `2026-07-28` has no session
   concept, and a stateless HTTP request has no stable connection identity.
2. **PID as the process handle.** Rejected: PIDs are reused; a stale PID can
   point to a different process.
3. **Singleton process state.** Rejected: hides state, breaks multi-client use
   and makes tests order-dependent.
4. **Return full state on every call.** Rejected: expensive and still does not
   solve concurrent mutation.

## References

- [MCP protocol](../mcp/protocol.md)
- [Architecture](../architecture.md)
- [MCP specification: 2026-07-28](https://modelcontextprotocol.io/specification/2026-07-28)
