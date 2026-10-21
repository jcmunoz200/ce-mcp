# Gap deferrals

## Purpose

The curated catalog has 81 tools. Seventy have an identified backend mapping
(`mapped`); eleven have no reliable mapping yet (`gap`). A gap is not a promise,
and a deferred gap is not coverage.

This page records the formal disposition of every remaining gap: the evidence
that is missing, why the tool is not mapped now, and the condition that would
close it. No tool on this page is implemented or tested.

## Summary

| Disposition | Count | Tools |
| --- | ---: | --- |
| Deferred to an F0 runtime experiment | 1 | `process_close` |
| Deferred to the native backend | 7 | six pointer-scan tools, `stacktrace` |
| Deferred pending form-automation design | 1 | `ui_open_tool` |
| Deferred pending snapshot design | 1 | `snapshot_restore` |
| Out of scope for the macOS release | 1 | `trainer_generate` |
| **Total** | **11** | |

These dispositions do not change the MCP contract. A gap can later become
`mapped` without changing its name, input schema or output schema.

## process_close

**Evidence.** `celua.txt` documents `openProcess(pid)` and
`openProcess(name)` but no `closeProcess`. The string `CloseProcess` appears
only in the low-level API-pointer list used by `setAPIPointer`. The convention
`openProcess(0)` is not documented and has not been tested.

**Why it is deferred.** Closing a process is destructive and easy to confuse
with `closeCE()`, which closes Cheat Engine rather than the target. Guessing the
semantics would be worse than reporting the gap.

**Closure criterion.** Run an F0 experiment against a disposable self-signed
target:

1. open the target and record `getOpenedProcessID()`;
2. call `openProcess(0)`;
3. verify the PID, process handle and dependent handles;
4. verify that memory and scan operations fail with the documented error;
5. repeat after reopening the same target.

If `openProcess(0)` closes the target, map `process_close` to it. If it does
not, implement the close in the native backend. Do not infer the behavior from
the GUI or from another Cheat Engine version.

## Pointer scan (six tools)

**Tools.** `pointer_scan_start`, `pointer_scan_status`, `pointer_scan_results`,
`pointer_scan_rescan`, `pointer_scan_cancel` and `pointermap_generate`.

**Evidence.** `celua.txt` has no direct pointer-scan API. It documents
`getPointerSize`/`setPointerSize` and pointer-map callbacks
(`onPointerMapGenerationStart` and `onPointerMapGenerationFinish`), but the
pointer scanner itself is orchestrated by Cheat Engine forms. The callbacks are
hooks, not a scan API.

**Why it is deferred.** The alternatives are a native scanner or form
automation. Form automation would depend on form names, control layout and
timing, and would be fragile across Cheat Engine versions. A native scanner is a
larger engineering project than the F0 bridge spike.

**Closure criterion.** Implement a native pointer-map and pointer-scan engine
with a defined result format, or a narrowly scoped form adapter that:

1. is guarded by an exact Cheat Engine version;
2. has a cancellation path;
3. returns structured results rather than screen text;
4. passes a known pointer-chain fixture on macOS arm64.

Until one of those exists, the tools remain gaps. They should be advertised as
unavailable in `ce_capabilities` rather than silently returning partial data.

## stacktrace

**Evidence.** There is no `getStack` or `getCallStack` function in `celua.txt`.
`TFrmTracer` exposes `Entry` and `StackEntry` data, but only through a tracer
form; there is no documented global getter for that form. The context APIs
return registers, not a call stack.

**Why it is deferred.** A reliable trace needs stack unwinding, frame
validation and module/symbol resolution. On arm64 this is native work, and the
existing arm64 context coverage is already an open question.

**Closure criterion.** A native unwinder that returns a bounded frame list with
module offsets, validated against a known call chain on a self-signed target.
The MCP result must mark unresolved frames rather than inventing symbols.

## ui_open_tool

**Evidence.** `celua.txt` documents `getFormCount()`, `getForm(index)`,
`getMainForm()`, `getMemoryViewForm()` and `getLuaEngine()`. `Form` exposes
`show()` and `showModal()`. There is no documented "open tool by name" call and
no stable menu-invocation API.

**Why it is deferred.** The tool could show an existing form, but it cannot
portably create or locate every Cheat Engine tool. Form names and class names
are implementation details. A partial allowlist would be useful, but it should
not be described as "open any tool".

**Closure criterion.** Define a small allowlist of stable forms with documented
getters, test show/raise/focus behavior, and return a structured
"form not available" error for everything else. If broader coverage is needed,
use native UI automation with explicit version guards. Do not scrape or
synthesize clicks in the F0 bridge.

## snapshot_restore

**Evidence.** `snapshot_create` has a proposed mapping (`saveTable(stream)` plus
`readBytes()`), but the snapshot format and storage policy are not designed.
`addSnapshotAsComment` creates an Auto Assembler comment containing original
code; it is not a runtime snapshot API. There is no documented restore call.

**Why it is deferred.** Restoring memory and table state raises questions that
the catalog has not answered: which regions are captured, how overlaps are
handled, what happens when the target changed, whether restoration is atomic,
and how a partial failure is reported. A restore that silently overwrites live
state would be unsafe.

**Closure criterion.** Define a versioned snapshot schema and a restore policy
covering region selection, table state, conflicts, atomicity and rollback. Then
compose `readBytes`/`writeBytes` and table loading, and test a failed restore
that must leave the target unchanged.

## trainer_generate

**Evidence.** The trainer generator is Windows/PE-centric. `celua.txt` exposes
`registerEXETrainerFeature` as an extension hook, not a documented
cross-platform "generate trainer" call. The tool definition already limits the
platform to Windows.

**Why it is deferred.** It is outside the macOS release and would require a
Windows backend plus PE packaging. It is not a macOS bridge gap.

**Closure criterion.** A Windows backend with an explicit trainer contract and
a reproducible output artifact. Keep it out of the macOS profile until then.

## Options considered and rejected

| Option | Why it was rejected |
| --- | --- |
| Assume `openProcess(0)` closes the target | Undocumented and destructive |
| Automate the pointer-scan and tracer forms in F0 | Fragile, hard to cancel, hard to test |
| Return a synthetic stack trace from registers | Would invent frames and symbols |
| Call `closeCE()` for `process_close` | Closes Cheat Engine, not the target process |
| Treat `snapshot_create` as proof that restore works | Create and restore are different operations |

## Reporting

- The coverage table remains the source of truth: **70 mapped, 11 gaps**.
- A gap is never reported as implemented or tested.
- Closing a gap requires a new mapping, a test result and a coverage update.
- If a gap cannot be closed without a native backend, record that dependency
  instead of leaving an open-ended promise.

See [Coverage](../reference/coverage.md) for the current counts and
[Tool migration](06-tool-migration.md) for the status definitions.
