# Tool migration

## Goal

Every documented Cheat Engine capability must be accounted for. "All tools"
does not mean one MCP tool per dialog. It means:

1. a curated MCP tool for the operation, or
2. a generated tool for the documented Lua function, or
3. an explicit exclusion with a reason and a platform note.

The coverage matrix is the contract. It is generated from the same definitions
as the server.

## Method

1. **Inventory.** Extract the documented Lua function names and the visible
   Cheat Engine domains.
2. **Domain model.** Group capabilities into process, memory, scan, table,
   symbols, pointer scan, assembly, debugger, structures, Lua, UI and misc.
3. **Curated design.** Define the tools an agent should see by default. Each
   tool gets a stable name, typed parameters and a backend mapping.
4. **Mapping.** For every Lua function, record whether it is covered by a
   curated tool, will be generated as its own tool, or is excluded.
5. **Coverage check.** CI compares the documented names with the catalog and
   fails on an unexplained gap.

## Curated catalog

The design contains **81 tools** in 13 groups.

| Group | Count | Tools |
| --- | --- | --- |
| Meta | 4 | `ce_info`, `ce_status`, `ce_capabilities`, `ce_handles` |
| Process | 7 | `process_list`, `process_open`, `process_close`, `process_info`, `module_list`, `region_list`, `thread_list` |
| Memory | 10 | `memory_read`, `memory_write`, `memory_read_value`, `memory_write_value`, `memory_read_string`, `memory_alloc`, `memory_free`, `memory_protect`, `memory_fill`, `memory_dump` |
| Scan | 8 | `scan_start`, `scan_next`, `scan_cancel`, `scan_status`, `scan_results`, `scan_refresh`, `scan_save`, `scan_load` |
| AOB/Symbols | 5 | `aob_scan`, `symbol_resolve`, `symbol_lookup`, `symbol_list`, `address_parse` |
| Table | 10 | `table_new`, `table_load`, `table_save`, `table_entries`, `table_add`, `table_update`, `table_delete`, `table_freeze`, `table_set_value`, `table_run_script` |
| Pointer scan | 6 | `pointer_scan_start`, `pointer_scan_status`, `pointer_scan_results`, `pointer_scan_rescan`, `pointer_scan_cancel`, `pointermap_generate` |
| Disassembly/Assembly | 4 | `disassemble`, `assemble`, `auto_assemble`, `code_cave_find` |
| Debugger | 12 | `debug_attach`, `debug_detach`, `debug_status`, `breakpoint_set`, `breakpoint_remove`, `breakpoint_list`, `debug_continue`, `debug_step`, `registers_get`, `registers_set`, `stacktrace`, `debug_wait` |
| Structures | 4 | `structure_new`, `structure_define`, `structure_read`, `structure_list` |
| Lua | 3 | `lua_eval`, `lua_call`, `lua_api_list` |
| UI/App | 5 | `ui_state`, `ui_open_tool`, `hotkey_list`, `hotkey_set`, `speedhack_set` |
| Misc | 3 | `trainer_generate`, `snapshot_create`, `snapshot_restore` |

The machine-readable definitions are in [`spec/tools/`](../../spec/tools/). The
catalog reference is [MCP tool catalog](../mcp/tool-catalog.md).

## Lua API coverage

`spec/lua-api.yaml` contains the 585 documented function names and a migration
status. The design has three coverage levels:

| Level | Meaning |
| --- | --- |
| `curated` | The function is behind a curated MCP tool with a stable contract |
| `generated` | The function gets its own tool under the `lua` profile |
| `excluded` | The function is not exposed, with a documented reason |

The `lua` profile is optional because listing 585 tools in every agent context
would be counterproductive. The `all` profile exists for clients that
explicitly want the full surface.

## Why this is the full surface

Cheat Engine has three kinds of capability:

1. **Documented automation.** The 585 Lua functions in `celua.txt`. These are
   covered by curated tools, generated tools, or an explicit exclusion.
2. **Operations with no Lua entry point.** Pointer scan, parts of structure
   dissect, trainer generation and some UI flows. These are listed as `gap` or
   excluded, not silently omitted.
3. **Dialogs and presentation.** Settings, forms, colors and layout. These are
   not separate tools. A setting becomes a parameter on the operation it
   controls; a form is not a capability by itself.

The catalog therefore accounts for the automation surface, names the native
gaps, and refuses to claim that a GUI dialog is a tool. That is what "all tools"
means in this project.

## Status vocabulary

| Status | Meaning |
| --- | --- |
| `designed` | Tool contract exists in `spec/`; no implementation |
| `mapped` | Backend mapping is identified; not tested |
| `implemented` | Backend call works in a development build |
| `tested` | Contract tests and an end-to-end test pass |
| `unsupported` | Not available on the platform or backend |
| `gap` | No reliable mapping exists yet |

Today no curated tool is implemented or tested. A tool with an identified
backend strategy is marked `mapped`; a tool without one is marked `gap`.

## Platform matrix

| Platform | Curated tools | Lua bridge | Notes |
| --- | --- | --- | --- |
| macOS | Designed | Planned | First target; arm64 is the primary test host |
| Windows | Designed | Future | Named pipes or localhost TCP; DBVM/kernel features remain out of scope |
| Linux | Designed | Future | The public source has Linux support; the bridge must be validated |

The `platforms` field in each tool definition is authoritative.

## Known limitations and gaps

| Limitation or gap | Reason | Candidate solution |
| --- | --- | --- |
| Pointer scan | No direct Lua API found in `celua.txt`; the UI forms orchestrate it | Native backend or controlled form automation |
| Structure dissect UI | The `structure_*` tools cover definitions and reads; the interactive window is form-driven | Use `structure_*`; UI parity is out of scope |
| arm64 debugger context | The binary contains explicit aarch64 context error paths | Native Mach exception backend |
| Code cave scanner | No single documented API; `code_cave_find` composes region and byte scans | Planned composed tool |
| Trainer generation | Windows/PE-centric | Out of scope for macOS; revisit later |
| DBVM and kernel features | Windows-only by design | Excluded |

The 11 tool-level gaps and their closure criteria are recorded in
[Gap deferrals](08-gap-deferrals.md). This table also includes limitations that
are not individual tools.

## Coverage in CI

When implementation starts, CI will:

1. parse the documented function names from a local Cheat Engine installation
   or a checked-in names file;
2. compare them with `spec/lua-api.yaml`;
3. fail if a name is missing, duplicated or has no status;
4. compare `spec/tools/*.yaml` with the generated server registration;
5. fail if a tool is documented but not registered, or vice versa;
6. publish `docs/reference/coverage.md` as a generated report.

The coverage report must distinguish **designed**, **implemented** and
**tested**. A design document is not coverage.

## Adding a tool

1. Add the definition to the right `spec/tools/*.yaml` file.
2. Assign a profile, platforms, backend and risk class.
3. Add the Cheat Engine Lua mapping or mark it `gap`.
4. Update `spec/lua-api.yaml` if a documented function becomes covered.
5. Run the generator and the coverage check.
6. Update the generated reference and coverage report.

See [CONTRIBUTING.md](../../CONTRIBUTING.md) for the review process.
