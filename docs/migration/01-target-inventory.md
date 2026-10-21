# Target inventory: Cheat Engine 7.7

**Evidence status:** shell observations plus a formal REA 6.7.0 CLI/Mach-O pass
on 2026-10-21. The pass used the built-in artifact, Mach-O, property-list and
code-signature providers. No Ghidra or Hopper disassembly was required, and no
runtime behavior was observed. No Cheat Engine binary or asset is included in
this repository.

## Identity

| Field | Value |
| --- | --- |
| Application | `/Applications/CheatEngine77.app` |
| Bundle name | Cheat Engine 7.7 |
| Bundle identifier | `org.cheatengine.cheatengine` |
| Version | `7.7` (`CFBundleShortVersionString`) |
| Executable | `Contents/MacOS/cheatengine` |
| Architectures | Universal: `x86_64` and `arm64` |
| Main executable minimum macOS | 10.15 (`LC_BUILD_VERSION`, SDK 10.15) |
| Bundled Lua minimum macOS | 13.3 (`LC_BUILD_VERSION`, SDK 13.3) |
| Info.plist minimum | No `LSMinimumSystemVersion` key |
| Bundle size | ~104 MB |
| Main binary size | 83,482,352 bytes |
| Main binary SHA-256 | `c52dbafcf101cea70c5fc3ccf3c8df704a55cc414b8480b717152a502b916de6` |
| Lua library | `Contents/MacOS/liblua53.dylib`, 597,904 bytes |
| Lua library SHA-256 | `2bade6b1645cbff550f5bd1499cd759cdfb211e2b82d1142748637eb9b647885` |
| Code signature | Developer ID Application: Eric Heijnen (`3W4HR5987G`) |
| Signature timestamp | 2026-05-29 09:43:47 |
| Hardened runtime | Yes |
| Entitlements | None observed |

The bundle is code-signed and uses universal Mach-O slices. The main executable
declares macOS 10.15, but the bundled `liblua53.dylib` declares macOS 13.3. If
that dylib is loaded on launch, 13.3 is the effective floor; this is an
**inference**, not a runtime test. The `Info.plist` does not declare a minimum.
This inventory did not assess notarization or Gatekeeper behavior.

## Bundle contents

| Path | Notes |
| --- | --- |
| `Contents/MacOS/cheatengine` | Main application |
| `Contents/MacOS/liblua53.dylib` | Lua 5.3 runtime |
| `Contents/MacOS/libspeedhack.dylib` | Speedhack library; only `i386` and `x86_64`, no `arm64` |
| `Contents/MacOS/libtcc-64.dylib` | Tiny C Compiler; universal `x86_64`/`arm64` |
| `Contents/MacOS/libtcc-arm64.dylib` | Tiny C Compiler; universal `x86_64`/`arm64` |
| `Contents/MacOS/tcclib/` | TCC runtime headers and license files |
| `Contents/MacOS/tutorial-aarch64.app` | arm64 tutorial target |
| `Contents/MacOS/tutorial-x86_64.app` | x86_64 tutorial target |
| `Contents/MacOS/include/` | C headers and source fragments, including `celib.h`, `cesocket.h`, `cepipelib.c` and `luaclient.c` |
| `Contents/Lua/` | `main.lua`, `defines.lua`, Autorun scripts, `celua.txt` |
| `Contents/Lua/Autorun/` | macOS scripts for speedhack, Mono/.NET, Java, pseudocode and symbol support |
| `Contents/Lua/Autorun/dylibs/libMonoDataCollectorMac.dylib` | Universal Mono data collector; minimum macOS 11.0 |
| `Contents/Lua/Extensions/AITools/` | Bundled MIT-licensed AI assistant extension; not part of the core Lua API |
| `Contents/Lua/Extensions/UETools/` | Bundled Unreal Engine inspection extension; not part of the core Lua API |
| `Contents/Languages/` | 49 language and translation files |

The `Lua/Extensions` and `Lua/Autorun` directories are relevant to loading a
bridge script without modifying the application binary. The exact loading
order, permissions and extension enablement have not been tested. The bundled
extensions are third-party scripts, not entries in `celua.txt`; see
[coverage](../reference/coverage.md) for the scope boundary.

## Formal REA pass

A formal REA 6.7.0 CLI/Mach-O pass was run on 2026-10-21. It used the built-in
artifact, Mach-O, plist and code-signature providers; no Ghidra/Hopper
disassembly and no target execution were involved.

Evidence references:

- artifact parent: `ev_c09daa54d23febdd0dc2644a9fb83c76686b78c9754f0f4ed4abe4d8ac9f2a2c`
- artifact inventory: `ev_9fbd8b477591cec873e8408c746462ca6fc93d2816cf7df7476ffc48ada323c5`
- main Mach-O: `ev_f8dcc9d49ffc55bfc936a11f08ae127fe08f1e7142904e07e7047c8b85f165db`
- `liblua53.dylib`: `ev_59b5676589f543d1cfc3dd8ff7e4b256a7ac3ab31ddd422a2955e4b73ea7ca26`
- code signature: `ev_138adc2cec7795d98d869835ae97d988ac45adc1ee7aea126fcf8bcc42b38c3f`
- `Info.plist`: `ev_760867c78a69d30853d7b4c747f5748bdae0ea45c9bed07ee25ae27c571f1b0e`
- dylib resolution: `ev_0d5bb1554db021ad57a9b22401f01ae543fad5e1e6aaf669e57159719ad2ddcd`

### Artifact graph

The artifact provider reported 254 graph nodes, 261 path occurrences and 260
edges. The root directory SHA-256 was
`16b0c557426e935d9b79e8c4193f3b8ee125b4689800f1ac08197329a7168374`; the
graph SHA-256 was
`d4cfcf575446f182f853ebdc348cf35936cb364a128e2e725d802d3d7617d37c`.
The graph contains eight Mach-O files: the main executable, four bundled
libraries, two tutorial executables and the Mono collector. It also contains
the C headers and source fragments under `Contents/MacOS/include`, and the
bundled Lua scripts under `Contents/Lua`.

### Mach-O facts

| Binary | Architectures | Minimum macOS | Notes |
| --- | --- | --- | --- |
| `cheatengine` | `x86_64`, `arm64` | 10.15 | SDK 10.15; LD 1167.5; UUID `205C584C-1289-347E-847C-323C73662D42` |
| `liblua53.dylib` | `x86_64`, `arm64` | 13.3 | SDK 13.3; LD 857.1; one dependency (`libSystem`) |
| `libspeedhack.dylib` | `i386`, `x86_64` | 10.5 (x86_64 slice) | No `arm64` slice |
| `libtcc-64.dylib` | `x86_64`, `arm64` | 11.0 | SDK 13.3 |
| `libtcc-arm64.dylib` | `x86_64`, `arm64` | 11.0 | SDK 13.3 |
| `libMonoDataCollectorMac.dylib` | `x86_64`, `arm64` | 11.0 | SDK 15.5 |

The main executable returned 125,814 export records and 511 import records.
The provider marks both collections as non-exhaustive, so these are lower
bounds. It has 33 load commands, 17 dynamic-library dependencies, five segments
and one entry point. The export count is in the same order as the 126,325
`nm` symbol lines; the two metrics are not identical.

The main binary imports Mach task and memory APIs (`task_for_pid`,
`task_suspend`, `task_resume`, `mach_vm_read_overwrite`, `mach_vm_write`,
`mach_vm_protect`, `mach_vm_region`), thread-state APIs and POSIX socket APIs
(`socket`, `bind`, `listen`, `accept`, `connect`, `recv`, `send`). It does not
import `ptrace`; the `ptrace` string observed earlier may come from a dynamic
lookup or an unused code path. **Inference, not a runtime observation.**

### Signature and plist

`inspect_signature` reported a valid signature with the authority chain
`Developer ID Application: Eric Heijnen (3W4HR5987G)` → `Developer ID
Certification Authority` → `Apple Root CA`, hardened runtime enabled, and no
entitlements returned. The signature timestamp was 2026-05-29 09:43:47. The
CDHash was `1f1316e5bbdc6ba7f7a1144a2f8fed36e14176b5`. The designated
requirement binds the identifier `org.cheatengine.cheatengine` to the developer
team. Notarization and Gatekeeper trust were not assessed.

`inspect_plist` read the XML `Info.plist`: `CFBundleName` is `Cheat Engine 7.7`,
`CFBundleIdentifier` is `org.cheatengine.cheatengine`, `CFBundleShortVersionString`
is `7.7`, and `CFBundleExecutable` is `cheatengine`. There is no
`LSMinimumSystemVersion` key.

### Bridge-relevant observations

`celua.txt` documents `LuaServerSocket` and
`createServerSocket(port, localhostOnly)`. It is a blocking TCP socket;
`acceptConnection` blocks and recommends a separate thread. `LuaPipe` exposes
`readBytes`, `writeBytes`, `Timeout`, `OnTimeout` and `OnError`. This supports
the planned bridge, but its behavior on macOS 26 remains untested.

The bundle also contains C source fragments for a named-pipe Lua client
(`luaclient.c`, `cepipelib.c`) and a `cesocket.h` that defines only `AF_UNIX`.
The planned bridge uses the documented Lua TCP socket instead of adding a native
C client. The `Lua/Autorun/SpeedhackV3Mac.lua` script shows that macOS speedhack
uses `compile` and API hooks; the bundled `libspeedhack.dylib` is not an arm64
library.

## Binary surface

| Metric | Value | How measured |
| --- | --- | --- |
| Total symbol lines (arm64 slice) | 126,325 | `nm -arch arm64` |
| FPC-mangled symbols (`$$_`) | 119,479 | `nm` + filter |
| Mach-O export records | 125,814 returned; non-exhaustive | REA `inspect-macho` |
| Mach-O import records | 511 returned; non-exhaustive | REA `inspect-macho` |
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

**Observation:** the application imports `AuthorizationCreate`,
`task_for_pid`, `mach_vm_*` and `thread_*_state`, and its strings contain the
SIP, root and `ptrace` messages. The formal import list does not contain a
`_ptrace` symbol; the string does not prove a `ptrace` call.

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

# Formal REA 6.7.0 pass (read-only, no target execution)
npx -y rea-agents@6.7.0 inspect-artifact "$TARGET" --format json
npx -y rea-agents@6.7.0 inspect-macho "$TARGET/Contents/MacOS/cheatengine" --format json
npx -y rea-agents@6.7.0 inspect-macho "$TARGET/Contents/MacOS/liblua53.dylib" --format json
npx -y rea-agents@6.7.0 inspect-plist "$TARGET" --format json
npx -y rea-agents@6.7.0 inspect-signature "$TARGET" --format json
npx -y rea-agents@6.7.0 trace-dylib-resolution "$TARGET" --architecture arm64 --format json
lipo -info "$TARGET/Contents/MacOS/libspeedhack.dylib"
vtool -show-build -arch x86_64 "$TARGET/Contents/MacOS/libspeedhack.dylib"
```

## Unknowns

- Which Lua functions actually work on macOS arm64; the documentation is
  cross-platform.
- Whether `createServerSocket` and `createThread` behave as required in the
  bridge on macOS 26.
- Whether the 13.3 minimum of `liblua53.dylib` prevents launch on an older
  macOS host that satisfies the 10.15 main-executable minimum.
- The exact behavior of `openProcess(0)` for closing a process.
- Whether the pointer scanner is reachable through the Lua API or only through
  its forms.
- Which debugger context fields are available on arm64.
- Whether the Lua/TCC speedhack path works on arm64 even though the bundled
  `libspeedhack.dylib` has no arm64 slice.
- The exact loading order, permissions and enablement of `Lua/Autorun` and
  `Lua/Extensions` when the application is code-signed.

These unknowns are carried into [the journal](journal/2026-10-21.md) and the
F0 spike.

## Evidence limits

This inventory does **not** establish:

- that every symbol is reachable or used;
- that every form is functional on macOS;
- that every documented Lua function is implemented in this binary;
- that the public source at 7.5 matches the 7.7 binary;
- runtime behavior of any feature;
- that the returned export and import records are complete.

The formal REA pass strengthens the artifact and Mach-O metadata evidence, but
it is static. It did not run the application, attach a debugger, decompile
functions with Ghidra/Hopper or test the Lua bridge. Those remain F0 work.
