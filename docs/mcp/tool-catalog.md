# Tool catalog

This is the human-readable catalog of the **81 curated tools**. The
machine-readable source of truth is [`spec/tools/`](../../spec/tools/).

> **Status:** every tool is designed; 70 have an identified backend mapping
> (`mapped`) and 11 have no reliable mapping yet (`gap`). None is implemented.
> See [Coverage](../reference/coverage.md).

The server also exposes `tool_search` as a built-in discovery tool in every
profile. It is not counted in the 81.

## Meta (4)

| Tool | Summary |
| --- | --- |
| `ce_info` | Server, build, Cheat Engine version and platform information |
| `ce_status` | Bridge state, active profile, open handles and capability summary |
| `ce_capabilities` | Machine-readable feature and coverage matrix |
| `ce_handles` | List, inspect and close backend handles |

## Process (7)

| Tool | Summary |
| --- | --- |
| `process_list` | List processes by PID and name, with an optional filter |
| `process_open` | Open a process by PID or name; returns a `process_handle` |
| `process_close` | Close a process handle and release its resources |
| `process_info` | PID, name, path, architecture, start time and handle state |
| `module_list` | List loaded modules with base, size and path |
| `region_list` | List memory regions with protection, state and type |
| `thread_list` | List thread IDs for the open process |

## Memory (10)

| Tool | Summary |
| --- | --- |
| `memory_read` | Read a byte range; returns hex or base64 |
| `memory_write` | Write a byte range |
| `memory_read_value` | Read a typed scalar (byte, int, float, double, pointer) |
| `memory_write_value` | Write a typed scalar |
| `memory_read_string` | Read an ASCII, UTF-8 or UTF-16 string |
| `memory_alloc` | Allocate memory in the target process |
| `memory_free` | Free a previous allocation |
| `memory_protect` | Change page protection for a range |
| `memory_fill` | Fill a range with a byte pattern |
| `memory_dump` | Dump a range to a file or return it in chunks |

## Scan (8)

| Tool | Summary |
| --- | --- |
| `scan_start` | First scan: exact, unknown, bigger/smaller, range, string, AOB |
| `scan_next` | Refine results: exact, changed, unchanged, increased, decreased |
| `scan_cancel` | Cancel a running scan or task |
| `scan_status` | Progress, result count and state |
| `scan_results` | Paginated addresses and current values |
| `scan_refresh` | Refresh the values of the current results |
| `scan_save` | Persist a scan session to the server workspace |
| `scan_load` | Restore a scan session |

## AOB and symbols (5)

| Tool | Summary |
| --- | --- |
| `aob_scan` | Array-of-bytes scan with wildcards |
| `symbol_resolve` | Resolve a symbol or CE address expression to an address |
| `symbol_lookup` | Map an address to a module and offset or symbol |
| `symbol_list` | List registered and user-defined symbols |
| `address_parse` | Parse `module+offset`, pointer and symbol expressions |

## Table (10)

| Tool | Summary |
| --- | --- |
| `table_new` | Create an empty table context |
| `table_load` | Load a `.CT` or `.CETRAINER` file into a table context |
| `table_save` | Save a table context to `.CT` |
| `table_entries` | Paginated address-list entries |
| `table_add` | Add a memory record with address, type and description |
| `table_update` | Update an entry's description, address, type or script |
| `table_delete` | Delete an entry |
| `table_freeze` | Freeze or unfreeze an entry |
| `table_set_value` | Set an entry's value |
| `table_run_script` | Activate an entry's Auto Assembler script |

## Pointer scan (6)

| Tool | Summary |
| --- | --- |
| `pointer_scan_start` | Start a pointer scan; returns a task ID |
| `pointer_scan_status` | Progress, result count and state |
| `pointer_scan_results` | Paginated pointer paths |
| `pointer_scan_rescan` | Rescan with a new target address |
| `pointer_scan_cancel` | Cancel a pointer scan |
| `pointermap_generate` | Generate a pointer map for later scans |

> **Gap:** `celua.txt` does not expose a direct pointer-scan API. These tools
> are designed but need a native backend or controlled form automation.

## Disassembly and assembly (4)

| Tool | Summary |
| --- | --- |
| `disassemble` | Disassemble instructions at an address |
| `assemble` | Assemble one or more instructions at an address |
| `auto_assemble` | Run an Auto Assembler script |
| `code_cave_find` | Find code caves in modules and regions |

## Debugger (12)

| Tool | Summary |
| --- | --- |
| `debug_attach` | Attach the debugger; returns a `debug_handle` |
| `debug_detach` | Detach and release breakpoints |
| `debug_status` | Debugger state, current thread and stop reason |
| `breakpoint_set` | Set a software, hardware or memory breakpoint |
| `breakpoint_remove` | Remove a breakpoint |
| `breakpoint_list` | List breakpoints and hit counts |
| `debug_continue` | Continue execution |
| `debug_step` | Step into, over or out |
| `registers_get` | Read registers for the current thread |
| `registers_set` | Write registers for the current thread |
| `stacktrace` | Return a stack trace with module offsets |
| `debug_wait` | Wait for a breakpoint or debug event with a timeout |

## Structures (4)

| Tool | Summary |
| --- | --- |
| `structure_new` | Create a structure definition |
| `structure_define` | Add fields with offsets, types and names |
| `structure_read` | Read a structure at an address |
| `structure_list` | List definitions in the current table |

## Lua (3)

| Tool | Summary |
| --- | --- |
| `lua_eval` | Evaluate Lua in Cheat Engine; returns a JSON-safe value |
| `lua_call` | Call a documented Lua function by name with typed arguments |
| `lua_api_list` | List documented functions and their migration status |

## UI and application (5)

| Tool | Summary |
| --- | --- |
| `ui_state` | Main-window state and open tools |
| `ui_open_tool` | Open a named Cheat Engine tool or form |
| `hotkey_list` | List registered hotkeys |
| `hotkey_set` | Register or update a hotkey |
| `speedhack_set` | Enable, disable and configure speedhack |

`speedhack_set` is mapped through `speedhack_setSpeed`, but the macOS arm64
Lua `compile` hook path is unverified; the bundled `libspeedhack.dylib` has no
arm64 slice.

## Misc (3)

| Tool | Summary |
| --- | --- |
| `trainer_generate` | Generate a trainer; Windows/PE-centric, out of scope for macOS |
| `snapshot_create` | Capture a server-side snapshot of a table and selected regions |
| `snapshot_restore` | Restore a snapshot |

## Profiles

| Profile | Tools | Contents |
| --- | --- | --- |
| `core` | 32 | The common process → memory → scan → table → Lua loop |
| `full` | 81 | All curated tools |
| `lua` | 589 | 4 meta tools + 585 generated Lua tools |
| `all` | 666 | 81 curated + 585 generated |

See [Profiles](profiles.md) for the exact `core` list.

## Generated Lua tools

The `lua` profile exposes one tool per documented Lua function:

```text
lua_readInteger
lua_writeInteger
lua_getAddressList
lua_createServerSocket
```

Generated tools:

- use the original function name after the `lua_` prefix;
- accept positional arguments in the documented order;
- return the function's return values as JSON;
- are marked `generated: true` in their metadata;
- are not a substitute for curated tools: they have no domain validation,
  pagination or risk-specific error messages.

The names and status live in [`spec/lua-api.yaml`](../../spec/lua-api.yaml).
Descriptions are not copied from Cheat Engine; the official document is linked
from [the Lua API reference](../reference/ce-lua-api.md).

## What is not in the catalog

- One tool per Cheat Engine dialog.
- DBVM and kernel-mode operations.
- Windows-only trainer internals on macOS.
- Anti-cheat bypass or stealth features.
- A GUI clone.

See [Tool migration](../migration/06-tool-migration.md) for the mapping method
and [Gap deferrals](../migration/08-gap-deferrals.md) for the remaining 11
tool-level gaps and their closure criteria.
