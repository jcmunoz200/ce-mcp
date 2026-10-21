# Profiles

## Why profiles

The complete surface is large:

- 81 curated tools;
- 585 documented Lua functions;
- 4 meta tools that are always useful.

Sending every schema to a model by default would waste context and increase
wrong-tool calls. Profiles make the trade-off explicit.

## Definitions

| Profile | Count | Contents | Default |
| --- | --- | --- | --- |
| `core` | 32 | Process, memory, scan, AOB/symbols, table and Lua essentials | Yes |
| `full` | 81 | Every curated tool | No |
| `lua` | 589 | 4 meta tools + 585 generated Lua tools | No |
| `all` | 666 | 81 curated + 585 generated | No |

`tool_search` is a built-in discovery tool available in every profile. It is not
counted in these totals.

## `core` (32)

The default workflow is find → filter → edit → save:

| Group | Tools |
| --- | --- |
| Meta (4) | `ce_info`, `ce_status`, `ce_capabilities`, `ce_handles` |
| Process (5) | `process_list`, `process_open`, `process_close`, `process_info`, `module_list` |
| Memory (5) | `memory_read`, `memory_write`, `memory_read_value`, `memory_write_value`, `memory_read_string` |
| Scan (6) | `scan_start`, `scan_next`, `scan_cancel`, `scan_status`, `scan_results`, `scan_refresh` |
| AOB/symbols (3) | `aob_scan`, `symbol_resolve`, `address_parse` |
| Table (7) | `table_new`, `table_load`, `table_save`, `table_entries`, `table_set_value`, `table_freeze`, `table_run_script` |
| Lua (2) | `lua_eval`, `lua_api_list` |

Not in `core`: region and thread enumeration, allocation, protection and memory
dumping, scan persistence, raw table editing, pointer scan, disassembly,
debugger, structures, UI and misc tools.

## `full` (81)

All curated tools. Use it when the agent needs the debugger, assembler,
structures or pointer scan.

The full list is in [Tool catalog](tool-catalog.md) and
[`spec/tools/`](../../spec/tools/).

## `lua` (589)

The 4 meta tools plus one generated tool per documented Lua function.

Use it when:

- a client needs the complete documented API;
- a script uses a function that has no curated tool;
- you are testing coverage.

Do not use it as the default. The schemas are numerous, generated tools have
weaker validation than curated tools, and many functions are platform-specific.

## `all` (666)

The union of `full` and the generated Lua tools. It is the literal "every tool"
mode. It is intended for compatibility testing and power users who understand
the context cost.

## Selecting a profile

Profiles are selected at server start, not per tool call, so `tools/list` is
stable and cacheable.

Planned CLI:

```bash
ce-mcp --profile core
ce-mcp --profile full --transport stdio
ce-mcp --profile all --transport http --bind 127.0.0.1 --port 7337
```

Planned environment variable:

```bash
CE_MCP_PROFILE=full ce-mcp
```

Planned MCP client configuration:

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

## Behavior outside the profile

- `tools/list` returns only the active profile plus `tool_search`.
- `tool_search` can find tools outside the profile and describe them.
- Calling a tool outside the profile returns `CE_TOOL_NOT_IN_PROFILE`.
- The error includes the profile that contains the tool and a remediation such
  as "restart with `--profile full`".
- The server does not add tools dynamically after startup. Stable lists are
  required for caching and prompt stability.

## Context budget

Approximate guidance, not a hard limit:

| Profile | Tool count | When to use |
| --- | --- | --- |
| `core` | 32 | Most agent sessions |
| `full` | 81 | Debugging, assembly, structures, pointer scan |
| `lua` | 589 | Full API compatibility, script migration |
| `all` | 666 | Testing, exhaustive automation, not normal chat |

The cost is not only the schema tokens. A long tool list also makes selection
harder. Start with `core` and switch only when the workflow requires it.

## Coverage and profiles

Coverage is reported per profile:

- `core` coverage: the 32 tools and their backend mappings;
- `full` coverage: the 81 curated tools;
- `lua` coverage: the 585 generated functions;
- `all` coverage: the union, with duplicates identified.

A function can be covered by a curated tool and also generated as a Lua tool.
This is intentional: the curated tool has a stable, validated contract; the
generated tool is the raw API.

## References

- [ADR-0004](../decisions/ADR-0004-tool-profiles.md)
- [Tool catalog](tool-catalog.md)
- [Coverage](../reference/coverage.md)
