# F0 bridge and MCP test plan

## Purpose

F0 is the first implementation milestone: a Lua bridge loaded into Cheat Engine
7.7 and a Rust MCP server over stdio. It is blocked until the implementation
license is chosen. This page defines the acceptance tests now, so the spike does
not invent its success criteria after the fact.

F0 exposes six tools:

- `ce_info`
- `process_list`
- `process_open`
- `memory_read`
- `scan_start`
- `scan_results`

Nothing on this page is implemented or tested.

## Preconditions

| Item | Requirement |
| --- | --- |
| License | An explicit implementation license has been chosen |
| Cheat Engine | 7.7 from `/Applications/CheatEngine77.app` |
| Host | macOS 26 on arm64 for the primary run |
| Target | A disposable, self-signed test process with known values |
| SIP | Record the host state; do not disable it for the first run |
| Permissions | Record whether the target is debuggable without elevation |
| Repository | No Cheat Engine assets, copied docs or target binaries committed |

The test target should contain a known integer, float, string and a small
allocated buffer. It must be safe to terminate and restart. Do not use a
third-party game or a process whose state matters.

## Bridge tests

### Loading and discovery

- Load `bridge/ce_bridge.lua` without modifying the application bundle.
- Confirm the bridge version, Cheat Engine version and Lua version in
  `ce_info`.
- Confirm `~/.ce-mcp/bridge.json` is created.
- Confirm the file mode is `0600`; record the owner and directory mode.
- Confirm the file contains a port, token and PID, and no secrets in logs.
- Restart Cheat Engine and confirm the bridge can be loaded again without a
  stale port or token.

### Protocol version compatibility

- Modern stdio: call `server/discover`, then `tools/list`, then one read.
- Legacy stdio: perform the `2025-11-25` `initialize` handshake, then
  `tools/list` and one read. Record whether the same tool schemas are returned.
- Send an unsupported version and verify `-32022` with the supported version
  list, not a silent downgrade.
- If the SDK does not expose a working legacy path on stdio, record it and
  narrow the compatibility claim instead of advertising an untested path.

### Network boundary

- Confirm the listener is bound to `127.0.0.1`, not `0.0.0.0` or `::`.
- Attempt a connection from another host or interface; it must fail.
- Attempt a connection with a missing token, an empty token and a wrong token;
  each must be rejected before Lua evaluation.
- Send a valid request after a rejected request; the bridge must remain usable.

### Framing and limits

- Send a valid length-prefixed JSON request and verify the response frame.
- Send a length prefix larger than the configured maximum; the bridge must
  reject it without allocating an unbounded buffer.
- Send a truncated frame, then close the socket; the bridge must recover.
- Send invalid UTF-8 or invalid JSON; return a structured error.
- Send an unknown method; return a structured error and keep the connection
  usable.
- Send two requests in one read; both must be processed in order.
- Send a partial frame across reads; it must be reassembled correctly.
- Record the actual behavior of `LuaPipe.Timeout` and `createServerSocket`; the
  documentation describes timeouts in seconds, but the bridge must verify it.

### Lifecycle and concurrency

- Disconnect a client while the bridge is idle; no thread or socket leak.
- Disconnect during a long scan; the scan must be cancellable or marked
  orphaned.
- Connect a second client; the documented behavior is one active client, or a
  clean `CE_BACKEND_BUSY`, not interleaved evaluation.
- Request shutdown; the bridge must close the listener and exit its thread.
- Restart the bridge and verify old process/scan handles are not silently
  reused.

## Tool tests

| Tool | Test | Pass condition |
| --- | --- | --- |
| `ce_info` | Call with no arguments | Returns server, Cheat Engine, Lua, OS and architecture versions |
| `ce_info` | Call with no target open | Still succeeds; no process-dependent fields are invented |
| `process_list` | List all processes | Returns bounded results with PID and name |
| `process_list` | Filter by exact and partial name | Filter is case-insensitive where documented; no duplicate PIDs |
| `process_list` | Paginate | Cursor is stable for an unchanged process list; invalid cursor is rejected |
| `process_open` | Open the test target by PID | Returns a process handle and correct PID |
| `process_open` | Open by name with one match | Returns the same target |
| `process_open` | Open an invalid PID or name | Returns `CE_PROCESS_NOT_FOUND`; no handle is leaked |
| `process_open` | Open a protected process | Returns a structured permission error; does not bypass SIP |
| `memory_read` | Read a known integer, float and string | Bytes match the target before and after the call |
| `memory_read` | Read an unmapped address | Returns `CE_INVALID_ADDRESS`; connection remains usable |
| `memory_read` | Request more than the size limit | Returns `CE_LIMIT_EXCEEDED`; no partial silent truncation |
| `memory_read` | Read after the target exits | Returns `CE_PROCESS_CLOSED` or the documented equivalent |
| `scan_start` | Scan for a known 4-byte integer | Returns a scan handle and task state |
| `scan_start` | Scan for a known float and string | Each value type follows its documented comparison and alignment |
| `scan_start` | Scan with an invalid range | Returns `CE_INVALID_ARGUMENT` |
| `scan_start` | Cancel a running scan | Returns `CE_SCAN_CANCELLED` and releases the scan thread |
| `scan_results` | Read results from a completed scan | Returns bounded, paginated results with addresses and values |
| `scan_results` | Read results from an unknown handle | Returns `CE_SCAN_NOT_FOUND` |
| `scan_results` | Read before the scan completes | Returns `CE_SCAN_BUSY` or a progress object, not partial garbage |
| `scan_results` | Paginate to the end | Last page has no duplicate or skipped addresses |

## Handle tests

- A `process_handle` from `process_open` is required by `memory_read` and
  `scan_start`.
- A `scan_handle` from `scan_start` is required by `scan_results`.
- Unknown, expired and cross-session handles return `CE_HANDLE_EXPIRED`.
- Closing a process invalidates its scan handles.
- Handle IDs are opaque and are not derived from target addresses.
- A handle from one MCP connection is not accepted by another unless the
  protocol explicitly supports that sharing; F0 should not support it.

## Error and security tests

- Every tool error uses the structured error object from
  [Errors](../mcp/errors.md).
- Token and absolute user paths do not appear in error messages or logs.
- The bridge does not evaluate Lua from unauthenticated clients.
- The bridge does not expose an arbitrary shell, filesystem or network proxy.
- The server does not install the bridge silently.
- The token file is not committed, printed or included in diagnostics.
- A malformed request cannot crash Cheat Engine; if it can, record it as a
  blocking defect.

## Evidence to capture

For every run, record:

1. the commit and bridge version;
2. the macOS version, architecture and SIP state;
3. the Cheat Engine version and binary SHA-256;
4. the test target identity and how it was built;
5. the exact request and response, with secrets redacted;
6. pass, fail or blocked, with the observed error;
7. timing for scan start, scan completion and a 1 MiB read;
8. whether elevation was required.

## Exit criteria

F0 is complete only when:

- all six tools satisfy the table above on macOS arm64;
- the modern and legacy stdio version paths behave as documented;
- the bridge survives malformed input, reconnects and restart;
- the localhost-only and token checks pass;
- the handle lifecycle and error mappings are demonstrated;
- no Cheat Engine binary, copied documentation or target binary is committed;
- the evidence is recorded in the journal and the coverage page is updated.

Pointer scan, stack traces, debugger controls, UI form opening, snapshot restore
and trainer generation are outside F0. Their dispositions are in
[Gap deferrals](08-gap-deferrals.md).

## Open questions this plan must answer

- Does `createServerSocket(port, true)` accept only loopback on macOS?
- What is the real timeout unit for `LuaPipe.Timeout`?
- Does `openProcess(0)` close the process, and is it safe to call twice?
- Which arm64 scan value types and alignments match the Lua documentation?
- Does loading from `Lua/Autorun` differ from loading through the Lua Engine?
- Does the code signature or hardened runtime restrict either load path?
