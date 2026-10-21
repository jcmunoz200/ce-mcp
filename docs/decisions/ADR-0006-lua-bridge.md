# ADR-0006: Use a Lua bridge as the first backend

- **Status:** Accepted
- **Date:** 2026-10-21
- **Supersedes:** none
- **Superseded by:** none

## Context

The MCP server needs a backend. The options are:

1. a native Rust implementation of memory access, scanning, debugging, assembly
   and Lua;
2. a Lua script running inside Cheat Engine, controlled over a local socket;
3. automation of Cheat Engine's GUI forms;
4. a plugin or injected library.

The target already contains Lua 5.3, `celua.txt` documents 585 functions, and
the API includes `createServerSocket`, `acceptConnection`, `createThread` and
`synchronize`. The binary also contains a `lua_server` unit, but its behavior
and protocol are undocumented.

## Decision

The first backend is a Lua script, `ce_bridge.lua`, loaded into Cheat Engine
7.7.

### Protocol sketch

```text
ce-mcp -> ce_bridge.lua
  connect 127.0.0.1:<port>
  send 4-byte big-endian length + UTF-8 JSON
  receive 4-byte big-endian length + UTF-8 JSON
```

Request:

```json
{
  "id": 1,
  "token": "...",
  "op": "call",
  "name": "readInteger",
  "args": ["0x1234"]
}
```

Response:

```json
{
  "id": 1,
  "ok": true,
  "result": 42
}
```

The bridge:

- binds to `127.0.0.1` only, using `createServerSocket(port, true)`;
- writes `{port, token, pid, version}` to `~/.ce-mcp/bridge.json` with mode
  `0600`;
- requires the token on every request before evaluating Lua;
- runs the accept loop in a `createThread` thread;
- uses `synchronize` for GUI-affecting operations;
- supports `ping`, `eval`, `call`, `cancel` and `shutdown`;
- encodes byte arrays as hex strings and rejects non-JSON-safe values;
- applies best-effort time and size limits.

The port is chosen by the bridge and written to the state file; the MCP server
does not scan a fixed port range. The token is random per launch.

The bridge is **opt-in**. The MCP server never installs or loads it silently.

## Consequences

### Positive

- Immediate coverage of the documented Lua API.
- Exact Cheat Engine semantics, including edge cases and bugs.
- The bridge is small enough to review and replace.
- The MCP contract can be validated before native work.
- No binary injection or code-signing changes to Cheat Engine.

### Negative

- Cheat Engine must be running and the bridge loaded.
- The bridge runs with Cheat Engine's privileges.
- Blocking Lua calls and GUI synchronization need careful handling.
- The socket protocol is another surface to secure.
- Performance is limited by Lua and JSON serialization.
- A crash in the bridge can affect Cheat Engine.

### Neutral

- A native backend can replace individual tools later.
- The bridge protocol is internal and may change before `v1.0`.
- The built-in `lua_server` is not used; its protocol is undocumented.

## Alternatives considered

1. **Native backend first.** Rejected: delays the MCP contract and duplicates a
   working engine.
2. **Use the built-in `lua_server`.** Rejected: undocumented protocol, unclear
   security model and no guarantee it is enabled on macOS.
3. **GUI automation.** Rejected: fragile, slow and inaccessible.
4. **Inject a native library into Cheat Engine.** Rejected: more invasive, code
   signing and stability risks, no need for the first backend.
5. **Named pipes only.** Deferred: TCP localhost is portable and supported by
   the Lua socket API; a pipe transport can be added later.

## References

- [Backend strategy](../migration/05-backend-strategy.md)
- [Protocol](../mcp/protocol.md)
- [Security](../mcp/security.md)
- [Target inventory](../migration/01-target-inventory.md)
