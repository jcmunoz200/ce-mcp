# Backend strategy

## Contract first

The MCP tools are the stable interface. They do not call Mach APIs, Lua or
Cheat Engine forms directly. They call a backend contract:

```rust
trait CheatEngineBackend {
    async fn execute(&self, op: Operation) -> Result<Value, BackendError>;
}
```

The `Operation` enum is the internal capability vocabulary. The MCP layer maps
validated tool arguments to operations; the backend maps operations to Cheat
Engine calls. This separation is what makes the migration possible.

## Stage 1: Lua bridge

The first backend is a Lua script loaded into Cheat Engine 7.7.

### Why

- Cheat Engine already implements memory access, scanning, the address list,
  debugger, assembler and Lua.
- The documented Lua API exposes most of those capabilities.
- The MCP design can be validated against a real application in weeks, not
  months.
- Cheat Engine semantics are preserved by construction, because Cheat Engine
  executes the operations.

### Shape

```text
ce-mcp (Rust)
  -> localhost TCP, length-prefixed JSON, token
  -> ce_bridge.lua (running inside Cheat Engine)
  -> Cheat Engine Lua API
  -> target process
```

The bridge:

- binds to `127.0.0.1` only;
- writes `{port, token, pid}` to a user-only file;
- authenticates before evaluating any code;
- accepts `ping`, `eval`, `call`, `cancel` and `shutdown`;
- runs the accept loop in a Lua thread;
- uses `synchronize` when an operation touches the GUI;
- returns JSON-safe values, with bytes encoded as hex or base64;
- enforces a best-effort execution timeout.

The wire format is a design proposal, not a tested protocol. F0 must validate
it.

### Coverage

The bridge can cover most of the documented API:

- process list and open;
- memory read/write, allocation, protection;
- first scan, next scan, unknown value, changed/increased/decreased;
- address list and memory records;
- `.CT` load/save;
- AOB scan, symbols, address parsing;
- disassembly, assembly, Auto Assembler;
- debugger breakpoints and context where the Lua API supports them; attach,
  stepping and stack traces are known gaps;
- structures and Lua evaluation;
- hotkeys, speedhack and some UI operations.

Known gaps are listed in [Tool migration](06-tool-migration.md).

## Stage 2: hybrid

Some tools have no reliable Lua API. They are marked `backend: native` and
implemented in Rust while the rest of the catalog remains on the bridge.

Likely first native tools:

- pointer scan orchestration;
- structure dissect beyond the Lua object model;
- region and module enumeration where the Lua output is insufficient;
- high-volume memory reads that would be inefficient through JSON;
- native debugger context fields on arm64.

The hybrid stage does not change tool names or semantics. A tool's `backend`
field changes from `lua` to `native` when its native implementation passes the
same contract tests.

## Stage 3: native

A full native backend is a separate, long-term project. It would need:

- `task_for_pid` and Mach VM access;
- region, module and thread enumeration;
- a scan engine with Cheat Engine-compatible value types and filters;
- a pointer scanner;
- Mach exception debugging, breakpoints and stepping;
- x86/x64 and arm64 disassembly and assembly;
- Auto Assembler compatibility;
- a Lua 5.3 runtime with the Cheat Engine API;
- `.CT` parsing and address-list semantics.

This is explicitly **out of scope** for the current roadmap. The MCP contract is
designed so that this work can happen incrementally without breaking clients.

## Compatibility promises

- Tool names, parameters and result shapes are stable within a major version.
- A backend swap must not change observable semantics.
- A tool that is unsupported by the current backend returns
  `CE_BACKEND_UNAVAILABLE` or `CE_UNSUPPORTED_PLATFORM`, not a silent no-op.
- Coverage is reported per backend, not hidden.

## Failure handling

| Failure | Behavior |
| --- | --- |
| Cheat Engine not running | `CE_BACKEND_UNAVAILABLE` with remediation |
| Bridge script not loaded | `CE_BACKEND_UNAVAILABLE` with setup instructions |
| Token mismatch | `CE_ACCESS_DENIED`; no Lua evaluation |
| Process exited | `CE_PROCESS_CLOSED`; handles invalidated |
| Scan cancelled | `CE_SCAN_CANCELLED`; partial results retained if requested |
| Lua error | `CE_LUA_ERROR` with the original message and traceback if available |
| Timeout | `CE_TIMEOUT`; the operation may still be running, so the tool documents whether it is safe to retry |

## Security

The bridge is arbitrary Lua execution by design. It is opt-in, localhost-only
and token-authenticated. The project does not attempt to sandbox Lua, because
that would diverge from Cheat Engine. See [Security](../mcp/security.md).

## Alternative considered: native first

A native-first backend would avoid the bridge but delay the MCP server until
memory access, scanning, the debugger and Lua are all functional. It also risks
designing the MCP contract against an incomplete engine. Rejected for the
current phase; recorded in
[ADR-0006](../decisions/ADR-0006-lua-bridge.md).
