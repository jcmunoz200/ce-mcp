# Source landscape

## Public repository

Cheat Engine publishes source code at
<https://github.com/cheat-engine/cheat-engine>.

Observed on 2026-10-21:

| Field | Value |
| --- | --- |
| Default branch | `master` |
| Latest release tag | `7.5` (2023-02-23) |
| Latest commit on `master` | 2025-04-19 |
| Files in the tree | 2,498 blobs |
| Pascal sources | 529 `.pas` |
| Lazarus forms | 164 `.lfm` |
| Program files | 20 `.lpr` |
| Lua files | 52 `.lua` |
| Root license file | Not present |

The repository contains a `tcclib` component with its own `COPYING` and
`RELICENSING` files. Those licenses apply to TCC, not to the whole project. The
absence of a root license means the licensing status of the Cheat Engine source
must not be assumed.

## What this means for the project

The shipped application is **7.7**. The newest public tag is **7.5**. The public
source is therefore useful as a reference, but it is not the specification for
the exact binary being targeted.

**Observation:** public source exists for 7.5 and earlier.

**Inference:** the 7.5 source explains most of the architecture, unit names and
Lua API, but not the 7.6/7.7 changes or the macOS-specific implementation in the
shipped 7.7 build.

## Rules for using the source

1. **Do not copy source code into this repository.** The project has no license
   for the implementation yet, and the upstream licensing status is unclear.
2. **Do not copy documentation prose.** Link to the official document and write
   original summaries.
3. **Use the source to understand structure, not as a binary specification.**
   Unit names, form names and API names are facts; implementation details may
   differ in 7.7.
4. **Prefer the shipped binary for behavior.** If the binary and the 7.5 source
   disagree, the binary wins.
5. **Record the version.** Every source claim should say whether it comes from
   the 7.5 tree or the 7.7 bundle.

## What the source is useful for

- Understanding the unit and form layout.
- Naming domains and tools consistently with Cheat Engine.
- Finding the Lua API surface and its overloads.
- Identifying which features are Windows-only by design.
- Designing a native backend later, with a clear map of responsibilities.

## What the source cannot answer

- The exact 7.7 macOS behavior.
- The arm64 debugger limitations.
- The speedhack implementation shipped in this bundle.
- The exact Lua bridge behavior in 7.7.
- Whether a documented function is compiled and reachable on macOS.

Those questions require the shipped binary, runtime observation or targeted
REA analysis.

## REA plan

The project intends to use REA for targeted evidence, not a full decompilation:

1. **Artifact and Mach-O inventory** with the built-in REA tools.
2. **Targeted native analysis** of macOS-specific units:
   `macexceptiondebuggerinterface`, `maccreateremotethread`, `macjitwriter`,
   `macumm`, `speedhack2` and `lua_server`.
3. **Runtime observation** of the bridge-relevant flows, once a test target and
   code-signing setup exist.
4. **Evidence ledger** with observation/inference/unknown labels.

This pass has **not** been run yet. The current inventory is shell-based. The
fact that a symbol or string exists is not proof that a feature works.

## Links

- Repository: <https://github.com/cheat-engine/cheat-engine>
- Latest public release: <https://github.com/cheat-engine/cheat-engine/releases/tag/7.5>
- Lua API document in the public tree:
  <https://github.com/cheat-engine/cheat-engine/blob/master/Cheat%20Engine/bin/celua.txt>
