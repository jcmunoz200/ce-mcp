# Errors

## Two layers

MCP distinguishes protocol errors from tool execution errors.

### Protocol errors

Use JSON-RPC error responses for:

- malformed JSON;
- unknown methods;
- unsupported protocol versions;
- invalid request metadata;
- unknown tools;
- invalid tool arguments that fail schema validation.

These are transport-level failures. The client should not retry without
changing the request.

### Tool errors

Use a successful JSON-RPC response with `isError: true` and a structured error
object in `structuredContent`:

```json
{
  "isError": true,
  "structuredContent": {
    "code": "CE_PROCESS_NOT_FOUND",
    "message": "No process matches 'game.exe'.",
    "remediation": "Call process_list and retry with a PID.",
    "retryable": false,
    "details": {
      "query": "game.exe"
    }
  }
}
```

A tool error means the tool ran and the operation failed. The transport is
healthy.

## Error object

| Field | Type | Required | Meaning |
| --- | --- | --- | --- |
| `code` | string | Yes | Stable machine-readable code |
| `message` | string | Yes | Human-readable summary |
| `remediation` | string | Yes | What the caller can do next |
| `retryable` | boolean | Yes | Whether an identical retry may succeed |
| `details` | object | No | Structured context, never secrets |
| `cause` | string | No | Underlying Cheat Engine or OS message |

Rules:

- `code` is stable across versions; do not parse `message`.
- `remediation` is actionable, not generic.
- `retryable` is about an **identical** retry. A retry with different arguments
  is a new request.
- `details` must not contain the bridge token, absolute user paths or raw target
  memory beyond what the caller requested.
- `cause` is sanitized and truncated.

## Codes

| Code | Meaning | Retryable | Typical remediation |
| --- | --- | --- | --- |
| `CE_INVALID_ARGUMENT` | Arguments are semantically invalid | No | Fix the argument; see `details.field` |
| `CE_UNSUPPORTED_PLATFORM` | Tool is not available on this OS/arch | No | Use a supported platform or a different tool |
| `CE_TOOL_NOT_IN_PROFILE` | Tool exists but is outside the active profile | No | Restart with the profile named in `details.profile` |
| `CE_BACKEND_UNAVAILABLE` | Bridge or Cheat Engine is not reachable | Yes | Start Cheat Engine and load the bridge |
| `CE_BACKEND_BUSY` | Bridge is processing another operation | Yes | Wait and retry; use tasks for long work |
| `CE_ACCESS_DENIED` | Token, permissions or OS access denied | No | Check the token file and macOS permissions |
| `CE_HANDLE_EXPIRED` | Handle is unknown, closed or expired | No | Re-open the process or start a new scan |
| `CE_PROCESS_NOT_FOUND` | No process matches the query | No | Call `process_list` |
| `CE_PROCESS_CLOSED` | The target exited or was detached | No | Re-open the process |
| `CE_NOT_ATTACHED` | No process is open for this handle | No | Call `process_open` |
| `CE_INVALID_ADDRESS` | Address is unmapped or malformed | No | Validate with `region_list` or `address_parse` |
| `CE_SCAN_NOT_FOUND` | Scan handle does not exist | No | Start a new scan |
| `CE_SCAN_BUSY` | A scan is already running for this handle | Yes | Wait, poll `scan_status`, or cancel |
| `CE_SCAN_CANCELLED` | Scan was cancelled | No | Start a new scan |
| `CE_DEBUG_NOT_ATTACHED` | No debugger session for this handle | No | Call `debug_attach` |
| `CE_BREAKPOINT_NOT_FOUND` | Breakpoint ID does not exist | No | Call `breakpoint_list` |
| `CE_LUA_ERROR` | Lua raised an error | No | Fix the script; see `cause` |
| `CE_TIMEOUT` | Operation exceeded its deadline | Maybe | Check status before retrying; it may still be running |
| `CE_INVALID_CURSOR` | Pagination cursor is invalid or expired | No | Re-run the original list call |
| `CE_LIMIT_EXCEEDED` | Result or size limit exceeded | No | Narrow the range or use pagination |
| `CE_INTERNAL` | Unexpected server error | Maybe | Report with the request ID |

## Retry guidance

- `retryable: true` means an identical request may succeed later. Use
  exponential backoff with jitter (for example 100 ms, 250 ms, 500 ms).
- `CE_TIMEOUT` is ambiguous. A scan may still be running. Poll `scan_status` or
  the task before retrying.
- `CE_BACKEND_BUSY` resolves when the current operation finishes; do not open a
  second bridge connection to bypass it.
- Never retry `CE_ACCESS_DENIED`, `CE_INVALID_ARGUMENT` or
  `CE_TOOL_NOT_IN_PROFILE` without changing something.
- Tasks are the preferred mechanism for operations that can exceed a client
  timeout.

## Mapping from the bridge

| Bridge condition | Code |
| --- | --- |
| Connection refused | `CE_BACKEND_UNAVAILABLE` |
| Missing or bad token | `CE_ACCESS_DENIED` |
| Lua `pcall` failure | `CE_LUA_ERROR` |
| `openProcess` returned no process | `CE_PROCESS_NOT_FOUND` |
| Read at unmapped address | `CE_INVALID_ADDRESS` |
| Scan thread still active | `CE_SCAN_BUSY` |
| Socket read timeout | `CE_TIMEOUT` |
| JSON encode failure | `CE_INTERNAL` |

## Progress and cancellation

- Long operations report progress through tasks or `scan_status`, not by
  emitting unstructured log lines.
- Cancellation is cooperative. `scan_cancel` and `tasks/cancel` set a flag; the
  scan checks it between regions.
- A cancelled operation returns `CE_SCAN_CANCELLED` and, if requested, the
  partial results.

## Logging

- stdio: logs go to stderr only.
- HTTP: logs go to stderr or a configured file.
- Audit logs are opt-in and separate from error logs.
- Tokens, raw memory and user paths are redacted by default.

## References

- [MCP tools](https://modelcontextprotocol.io/specification/2026-07-28/server/tools)
- [Security](security.md)
- [Tool catalog](tool-catalog.md)
