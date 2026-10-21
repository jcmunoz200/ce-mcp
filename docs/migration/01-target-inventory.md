# Target inventory: Cheat Engine 7.7

**Evidence status:** shell observations from the shipped macOS bundle. A formal
REA CLI + Mach-O pass has not been run yet. No Cheat Engine binary or asset is
included in this repository.

## Identity

| Field | Value |
| --- | --- |
| Application | `/Applications/CheatEngine77.app` |
| Bundle name | Cheat Engine 7.7 |
| Bundle identifier | `org.cheatengine.cheatengine` |
| Version | `7.7` (`CFBundleShortVersionString`) |
| Executable | `Contents/MacOS/cheatengine` |
| Architectures | Universal: `x86_64` and `arm64` |
| Minimum macOS | 10.15 (`LC_BUILD_VERSION`) |
| Bundle size | ~104 MB |
| Main binary size | 83,482,352 bytes |
| Main binary SHA-256 | `c52dbafcf101cea70c5fc3ccf3c8df704a55cc414b8480b717152a502b916de6` |
| Lua library | `Contents/MacOS/liblua53.dylib`, 597,904 bytes |
| Lua library SHA-256 | `2bade6b1645cbff550f5bd1499cd759cdfb211e2b82d1142748637eb9b647885` |

The bundle is code-signed and uses a universal Mach-O. The signature has the
hardened-runtime flag. This inventory did not assess notarization or Gatekeeper
behavior.

## Bundle contents

| Path | Notes |
| --- | --- |
| `Contents/MacOS/cheatengine` | Main application |
| `Contents/MacOS/liblua53.dylib` | Lua 5.3 runtime |
| `Contents/MacOS/libspeedhack.dylib` | Speedhack library; strings reference `/usr/local/lib/libspeedhack.dylib` |
| `Contents/MacOS/libtcc-64.dylib` | Tiny C Compiler, x86_64 |
| `Contents/MacOS/libtcc-arm64.dylib` | Tiny C Compiler, arm64 |
| `Contents/MacOS/tcclib/` | TCC runtime headers and license files |
| `Contents/MacOS/tutorial-aarch64.app` | arm64 tutorial target |
| `Contents/MacOS/tutorial-x86_64.app` | x86_64 tutorial target |
| `Contents/MacOS/include/` | C headers, including `celib.h` and `cesocket.h` |
| `Contents/Lua/` | `main.lua`, `defines.lua`, Autorun scripts, `celua.txt` |
| `Contents/Languages/` | 49 language and translation files |

The bundle also contains a `Lua/Extensions` directory and a `Lua/Autorun`
directory. Both are relevant to loading a bridge script without modifying the
application binary, but the exact loading behavior has not been tested yet.

## Binary surface

| Metric | Value | How measured |
| --- | --- | --- |
| Total symbol lines (arm64 slice) | 126,325 | `nm -arch arm64` |
| FPC-mangled symbols (`$$_`) | 119,479 | `nm` + filter |
| Unit-like prefixes | 10,393 | Symbol prefixes; includes LCL/RTL |
| `Tfrm*` classes | 136 | `strings` + filter |
| `TPF0` form resources | 344 | `strings` count |
| Documented Lua functions | 585 unique, 613 signatures | `celua.txt` |
| `celua.txt` size | 4,444 lines, ~240 KB | `wc`, `ls` |

The unit count includes the Lazarus Component Library, the Free Pascal runtime
and third-party units. It is not a count of Cheat Engine-specific modules.

The form count is a lower bound: it counts distinct `Tfrm*` strings, not every
form instance or frame. The `TPF0` count includes forms, frames and data
modules.

## Lua surface

`Contents/Lua/celua.txt` documents the Cheat Engine-specific Lua API. It is the
primary compatibility target for the bridge.

- 585 unique function names;
- 613 signatures, because some functions are overloaded;
- 22 unique `debug_*` functions (24 signatures), some of which are
  platform-specific;
- 39 unique `dbvm_*` and 19 unique `dbk_*` functions (43 and 20 signature
  lines), mostly Windows/DBVM-specific;
- the rest cover memory, scans, the address list, symbols, assembly, structures,
  files, forms, threads and settings.

An initial regex count of 586 included `ModuleLoader(Inheritance: -)`, which is
a class heading, not a function. The corrected function count is 585. The
signature count remains 613 because some functions are overloaded.

The names and migration status are recorded in
[`spec/lua-api.yaml`](../../spec/lua-api.yaml). Descriptions are intentionally
not copied from Cheat Engine's documentation; the original document is linked
from [the Lua API reference](../reference/ce-lua-api.md).

## macOS-specific units observed in symbols

These unit names indicate macOS-specific implementations that a native backend
would eventually need to understand:

| Unit | Relevance |
| --- | --- |
| `macexceptiondebuggerinterface` | Mach exception-based debugger |
| `maccreateremotethread` | Remote thread creation |
| `macjitwriter` | JIT/code writing |
| `macumm` | User-mode memory manager UI |
| `macPipe` | Pipe transport |
| `coresymbolication` | Symbolication via CoreSymbolication |
| `machotkeys` | Global hotkeys |
| `speedhack2` | Speedhack implementation |
| `lua_server` | Built-in Lua server |
| `tcclib` | TCC integration |

The binary also contains strings for `MacExceptionDebugInterface`,
`TMacExceptionDebugInterface`, `GetThreadContextArm64` and
`SetThreadContextArm64`. It contains error strings for
`GetThreadContext`/`SetThreadContext` on `aarch64`. This suggests that some
debugger context paths on Apple Silicon are incomplete or conditional.
**Inference, not a runtime observation.**

## Privilege and debugging model

Observed strings include:

- `"Have you disabled 'System Integrity Protection'(SIP) yet?"`
- `AuthorizationCreate`
- `already root` / `not root yet`
- `task_for_pid`
- `mach_vm_read` / `mach_vm_write` / `mach_vm_protect`
- `thread_get_state` / `thread_set_state`
- `ptrace`

**Observation:** the application contains code paths for privilege elevation,
Mach memory access and debugging.

**Inference:** full debugging of protected processes will require the same
macOS permissions as Cheat Engine: root or a debuggable target, and in some
cases SIP disabled. The MCP server cannot and will not bypass these
constraints. The F0 spike must measure what works on macOS 26 with SIP enabled
for a self-signed target.

## Reproduction commands

These commands produced the inventory. They are read-only.

```bash
TARGET="/Applications/CheatEngine77.app"
file "$TARGET/Contents/MacOS/cheatengine"
lipo -info "$TARGET/Contents/MacOS/cheatengine"
shasum -a 256 "$TARGET/Contents/MacOS/cheatengine"
nm -arch arm64 "$TARGET/Contents/MacOS/cheatengine" | wc -l
nm -arch arm64 "$TARGET/Contents/MacOS/cheatengine" | grep -c '\$\$_'
strings -a -n 5 "$TARGET/Contents/MacOS/cheatengine" | grep -E '^Tfrm' | sort -u | wc -l
strings -a -n 4 "$TARGET/Contents/MacOS/cheatengine" | grep -c 'TPF0'
grep -oE '^[A-Za-z_][A-Za-z0-9_]*\(' "$TARGET/Contents/Lua/celua.txt" \
  | sed 's/($//' | grep -v '^ModuleLoader$' | sort -u | wc -l
grep -E '^[A-Za-z_][A-Za-z0-9_]*\(' "$TARGET/Contents/Lua/celua.txt" \
  | grep -v '^ModuleLoader(' | wc -l
```

## Unknowns

- Which Lua functions actually work on macOS arm64; the documentation is
  cross-platform.
- Whether `createServerSocket` and `createThread` behave as required in the
  bridge on macOS 26.
- The exact behavior of `openProcess(0)` for closing a process.
- Whether the pointer scanner is reachable through the Lua API or only through
  its forms.
- Which debugger context fields are available on arm64.
- The exact loading order and permissions of `Lua/Autorun` and
  `Lua/Extensions` when the application is code-signed.

These unknowns are carried into [the journal](journal/2026-10-21.md) and the
F0 spike.

## Evidence limits

This inventory does **not** establish:

- that every symbol is reachable or used;
- that every form is functional on macOS;
- that every documented Lua function is implemented in this binary;
- that the public source at 7.5 matches the 7.7 binary;
- runtime behavior of any feature.

A formal REA pass can strengthen the evidence with Mach-O inventories and
targeted native analysis. It has not been run yet.
