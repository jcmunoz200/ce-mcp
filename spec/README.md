# Specification data

This directory is the machine-readable source of truth for the tool catalog. It
is data, not executable code.

## Files

| File | Contents |
| --- | --- |
| `profiles.yaml` | Profile definitions and counts |
| `tools/meta.yaml` | Meta tools |
| `tools/process.yaml` | Process and module tools |
| `tools/memory.yaml` | Memory tools |
| `tools/scan.yaml` | Scan tools |
| `tools/aob_symbols.yaml` | AOB, symbol and address tools |
| `tools/table.yaml` | Address-list and `.CT` tools |
| `tools/pointer.yaml` | Pointer scan tools |
| `tools/assembly.yaml` | Disassembly and assembly tools |
| `tools/debug.yaml` | Debugger tools |
| `tools/structures.yaml` | Structure tools |
| `tools/lua.yaml` | Lua tools |
| `tools/ui.yaml` | UI, hotkey and speedhack tools |
| `tools/misc.yaml` | Trainer and snapshot tools |
| `lua-api.yaml` | Documented Lua function names and migration status |
| `contracts/f0.yaml` | Input and output schemas for the six F0 tools |

## Tool schema

```yaml
- name: process_open
  summary: "Open a process by PID or name."
  profiles: [core, full]
  platforms: [macos, windows, linux]
  backend: lua
  ce_lua: "openProcess(pid) / openProcess(name)"
  risk: read
  status: mapped
  notes: "Returns a process_handle. Verify detach behavior in F0."
```

| Field | Required | Meaning |
| --- | --- | --- |
| `name` | Yes | Stable `snake_case` MCP tool name |
| `summary` | Yes | One-line description |
| `profiles` | Yes | `core`, `full`; meta tools also list `lua` and `all` |
| `platforms` | Yes | Platforms where the tool is intended to work |
| `backend` | Yes | `lua`, `native`, `server` or `mixed` |
| `ce_lua` | No | Cheat Engine Lua mapping, or `null` |
| `risk` | Yes | `meta`, `read`, `write`, `execute`, `debug` or `ui` |
| `status` | Yes | Mapping and implementation status |
| `notes` | No | Caveats, gaps or verification tasks |

## Status vocabulary

| Status | Meaning |
| --- | --- |
| `designed` | Contract only; no backend mapping yet |
| `mapped` | A backend mapping is identified; not implemented |
| `gap` | No reliable mapping yet |
| `implemented` | Backend call works in a development build |
| `tested` | Contract and end-to-end tests pass |
| `unsupported` | Not available on the listed platform |

A tool can be `mapped` and still be a design. `mapped` does not mean
implemented. The coverage report counts implemented and tested separately.

## F0 contracts

`contracts/f0.yaml` is the source of truth for the six F0 tool schemas. It uses
JSON Schema 2020-12 and mirrors the MCP `tools/list` fields: `name`,
`description`, `inputSchema`, `outputSchema` and annotations. The conventions,
limits and test obligations are explained in
[F0 tool contracts](../docs/mcp/f0-contracts.md).

## Counts

- Curated tools: 81.
- `core`: 32.
- `full`: 81.
- `lua`: 4 meta + 585 generated = 589.
- `all`: 81 curated + 585 generated = 666.
- `tool_search`: a built-in discovery tool, not counted.

## Validation

The future generator will validate:

- unique tool names;
- every tool has a profile and a status;
- every `core` tool is also in `full`;
- the counts match `profiles.yaml`;
- every `gap` has a note;
- every F0 tool has an input and output schema that parses as JSON Schema
  2020-12 and matches a tool name in `spec/tools/`;
- every `ce_lua` mapping is a known function or explicitly composed;
- no Cheat Engine assets or copied descriptions are present.

Until the generator exists, these files are reviewed by hand.
