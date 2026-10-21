# Coverage

## Honest status

There is **no implementation** yet.

| Metric | Value |
| --- | --- |
| Curated tools designed | 81 |
| Curated tools implemented | 0 |
| Curated tools tested | 0 |
| Documented Lua functions | 585 |
| Lua functions mapped to a curated tool | 0 verified |
| Lua functions implemented as generated tools | 0 |

"Designed" means a contract exists in `spec/`. It does not mean the tool works.
This page must never report design as coverage.

## Curated tools

| Group | Designed | Mapped | Gap | Implemented | Tested |
| --- | ---: | ---: | ---: | ---: | ---: |
| Meta | 4 | 4 | 0 | 0 | 0 |
| Process | 7 | 6 | 1 | 0 | 0 |
| Memory | 10 | 10 | 0 | 0 | 0 |
| Scan | 8 | 8 | 0 | 0 | 0 |
| AOB/symbols | 5 | 5 | 0 | 0 | 0 |
| Table | 10 | 10 | 0 | 0 | 0 |
| Pointer scan | 6 | 0 | 6 | 0 | 0 |
| Disassembly/assembly | 4 | 4 | 0 | 0 | 0 |
| Debugger | 12 | 8 | 4 | 0 | 0 |
| Structures | 4 | 4 | 0 | 0 | 0 |
| Lua | 3 | 3 | 0 | 0 | 0 |
| UI/application | 5 | 4 | 1 | 0 | 0 |
| Misc | 3 | 1 | 2 | 0 | 0 |
| **Total** | **81** | **67** | **14** | **0** | **0** |

A "mapped" tool has an identified Lua or native strategy. A "gap" has no
reliable mapping yet. Gaps are documented, not hidden.

### The 14 gaps

| Tool | Why it is a gap |
| --- | --- |
| `process_close` | No `closeProcess` in `celua.txt`; `openProcess(0)` behavior is unverified |
| `pointer_scan_start` | No direct Lua pointer-scan API |
| `pointer_scan_status` | Depends on pointer scan |
| `pointer_scan_results` | Depends on pointer scan |
| `pointer_scan_rescan` | Depends on pointer scan |
| `pointer_scan_cancel` | Depends on pointer scan |
| `pointermap_generate` | No direct Lua API; pointer-map callbacks exist, but orchestration is form-driven |
| `debug_attach` | No documented attach call; macOS arm64 behavior is unverified |
| `debug_detach` | No documented detach call |
| `debug_step` | Depends on arm64 context support |
| `stacktrace` | Depends on the debugger backend and symbolication |
| `ui_open_tool` | Form automation is possible but fragile; not designed yet |
| `snapshot_restore` | Restoring memory and table state needs a conflict and safety model |
| `trainer_generate` | Windows/PE-centric; out of scope for macOS |

## Lua API

| Family | Functions | Status on macOS | Notes |
| --- | ---: | --- | --- |
| `dbvm_*` | 39 | Unsupported on macOS | Windows/DBVM kernel features |
| `dbk_*` | 19 | Unsupported on macOS | Windows kernel features |
| `debug_*` | 22 | Unverified | Some functions are Windows-only; arm64 support is uncertain |
| Other | 505 | Unmapped | Cross-platform or unknown; mapping not started |
| **Total** | **585** | | |

The exact names and statuses are in [`spec/lua-api.yaml`](../../spec/lua-api.yaml).
`celua.txt` has 585 unique names and 613 signatures because some functions are
overloaded. An initial count of 586 included `ModuleLoader(Inheritance: -)`, a
class heading rather than a function.

The `lua` profile will generate one tool per documented function. Generated
tools are **designed**, not implemented. The `all` profile is the union of the
curated catalog and the generated tools.

### Bundled Lua code outside the count

Cheat Engine ships Lua code that is not part of `celua.txt`:

- `Contents/Lua/Autorun/`: speedhack, Mono/.NET, Java, pseudocode and symbol
  scripts loaded at startup;
- `Contents/Lua/Extensions/AITools/`: an MIT-licensed AI assistant extension;
- `Contents/Lua/Extensions/UETools/`: MIT-licensed Unreal Engine inspection
  tools.

These scripts are not in the 585-function count, are not generated as tools and
are not exposed by default. They are shipped application assets, not copied into
this repository. Adopting one later would require an explicit tool contract and
a license review.

## Platform coverage

| Platform | Curated design | Bridge | Native gaps |
| --- | --- | --- | --- |
| macOS arm64 | Yes | Planned; first target | Pointer scan, arm64 debugger context, interactive structure dissect, arm64 speedhack path unverified |
| macOS x86_64 | Yes | Planned | Same, plus x86-specific debugger behavior |
| Windows | Yes | Future | DBVM/kernel features excluded |
| Linux | Yes | Future | Bridge needs validation |

No platform is implemented yet.

## How coverage will be measured

1. Extract function names from `celua.txt`.
2. Compare with `spec/lua-api.yaml`.
3. Fail on missing, duplicate or status-less names.
4. Compare `spec/tools/*.yaml` with generated server registration.
5. Fail on a documented tool that is not registered, or a registered tool that
   is not documented.
6. Run contract tests against a real Cheat Engine 7.7 instance.
7. Publish the result here.

A function is only "covered" when:

- a curated tool implements it and has a contract test, or
- a generated tool is registered and tested, or
- it is explicitly excluded with a platform reason.

## How to update this page

- Do not edit the counts by hand once the coverage generator exists.
- Until then, update the table when a tool moves from `gap` to `mapped`,
  `implemented` or `tested`.
- Keep the gaps list current.
- Add a journal entry when a gap is closed or a new one is found.

## Related

- [Tool migration](../migration/06-tool-migration.md)
- [Tool catalog](../mcp/tool-catalog.md)
- [Lua API reference](ce-lua-api.md)
- [`spec/lua-api.yaml`](../../spec/lua-api.yaml)
