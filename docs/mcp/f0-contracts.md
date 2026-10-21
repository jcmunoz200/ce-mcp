# F0 tool contracts

## Purpose

The F0 spike exposes six tools:

- `ce_info`
- `process_list`
- `process_open`
- `memory_read`
- `scan_start`
- `scan_results`

Their machine-readable contracts are in
[`spec/contracts/f0.yaml`](../../spec/contracts/f0.yaml). This page explains the
conventions and the parts that are easy to implement inconsistently.

The contracts are designs, not implementation claims. The server generator will
turn them into the `tools/list` entries: `name`, `description`, `inputSchema`,
`outputSchema` and annotations. Until F0 passes, no tool is implemented or
tested.

## Contract conventions

| Topic | Rule |
| --- | --- |
| JSON Schema | 2020-12; every object rejects undeclared properties |
| Output | `structuredContent` must match `outputSchema`; `content` is a human-readable fallback |
| Addresses | Lowercase hex strings with `0x`, for example `0x100000000` |
| Handles | Opaque strings such as `ph_...` and `sh_...`; never parse them |
| PIDs and sizes | JSON integers; sizes and limits reject negative or zero values |
| Memory values | Strings, so an `int64` cannot lose precision in JSON |
| Scan values | Decimal strings for numeric types; raw text for string scans |
| Process filters | `process_list.filter` is a case-insensitive substring match |
| Process names | `process_open.name` is an exact, case-sensitive match |
| Bytes | Lowercase hex without separators, or standard base64 |
| Pagination | `next_cursor` is opaque and tied to the query that produced it |
| Errors | `isError: true` with a structured error object; see [Errors](errors.md) |

`process_open` accepts exactly one of `pid` or `name`; the schema uses `oneOf` to
reject both or neither. `name` is an exact process name. A name with multiple
matches is an error, not an arbitrary choice.

`scan_start` supports two comparisons in F0: `exact` and `unknown`. The other
Cheat Engine comparisons are deliberately absent until they are tested, because
they imply different `value` and `previous_value` semantics. Adding enum values
later is backward-compatible; guessing now is not.

`scan_start` may return `state: "running"` with a `task_id` when the tasks
extension is active. `scan_results` then reports progress without inventing
partial rows. A short scan may return `state: "completed"` directly.

## Tool summary

| Tool | Input | Output | Read-only |
| --- | --- | --- | --- |
| `ce_info` | No arguments | Server, Cheat Engine, bridge, Lua and host versions | Yes |
| `process_list` | `filter`, `limit`, `cursor` | `processes`, `next_cursor` | Yes |
| `process_open` | Exactly one of `pid`, `name` | `process_handle`, PID, name, architecture | No |
| `memory_read` | `process_handle`, `address`, `size`, `encoding` | `address`, `size`, `encoding`, `data` | Yes |
| `scan_start` | Process handle, value type, comparison, optional value and range | `scan_handle`, state, count, optional task | No |
| `scan_results` | `scan_handle`, `cursor`, `limit`, `include_values` | State, value type, rows, cursor, total | Yes |

The annotations in the contract mark only the read-only operations as
`readOnlyHint: true`. `process_open` and `scan_start` change server and target
state, so they are not read-only even though they do not write target memory.

## Limits and failure behavior

| Tool | Limit or rule |
| --- | --- |
| `process_list` | 1–1000 rows per page; an unchanged process list keeps a cursor stable |
| `process_open` | One process at a time, matching Cheat Engine's opened-process model |
| `memory_read` | 1 byte–1 MiB per call; no silent truncation |
| `scan_start` | 1–1,000,000 retained results; range and alignment are optional |
| `scan_results` | 1–1000 rows per page; `include_values: false` returns null values |
| Cursors | Invalid or expired cursors return `CE_INVALID_CURSOR` |
| Handles | Unknown or expired handles return `CE_HANDLE_EXPIRED` |
| Busy scans | `scan_results` returns `CE_SCAN_BUSY` rather than partial garbage |

A read at an unmapped address returns `CE_INVALID_ADDRESS`. A target that exits
returns `CE_PROCESS_CLOSED`, and its process and scan handles are invalidated.
The bridge may return `CE_TIMEOUT` if the operation exceeds its deadline; the
caller must poll the handle or task before retrying.

## Why these choices

**Addresses and values are strings.** JSON numbers cannot represent every
64-bit address or integer exactly, and a single schema that sometimes returns a
number and sometimes a string is harder for an agent to use. Strings are stable
across languages and transports.

**Handles are opaque.** A handle is not a PID and not an address. Clients must
not construct one, cache it across a server restart or send it to another
server. This is what keeps the MCP layer stateless while Cheat Engine remains
stateful.

**`memory_read` has a strict size cap.** Returning an unbounded byte array over
a local bridge invites memory spikes and truncated results. The cap makes the
limit explicit; `memory_dump` is the tool for large ranges, and it is outside F0.

**`scan_start` does not hide a long scan behind a blocked request.** The
contract allows a completed result or a task. The client must handle both
shapes. F0 may implement the simple completed path first, but the schema already
leaves room for the task extension.

## Test obligations

The [F0 test plan](../migration/09-f0-test-plan.md) is the acceptance test. The
contract adds these obligations:

1. Validate every successful result against its `outputSchema`.
2. Validate that undeclared input properties are rejected.
3. Exercise `process_open` with `pid` only, `name` only, both and neither.
4. Exercise `scan_start` with `exact` and `unknown`; reject other comparisons.
5. Verify that `memory_read` returns exactly `size` bytes and never truncates.
6. Verify that `next_cursor` is rejected when reused with different arguments.
7. Verify that `include_values: false` does not return stale values.
8. Verify that handles from a restarted server are rejected.

These tests do not exist yet. They are part of the implementation gate, not a
claim about the current repository.

## Future tools

The remaining 75 curated tools and the generated Lua tools need their own
contracts. They must follow the same conventions: opaque handles, string
addresses, bounded pages, structured errors and explicit output schemas. Adding
a tool must not change an existing tool's schema within a major version.
