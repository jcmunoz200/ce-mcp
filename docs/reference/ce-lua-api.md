# Cheat Engine Lua API

## What it is

Cheat Engine ships a Lua 5.3 runtime and documents its application-specific API
in `Contents/Lua/celua.txt`.

Observed in the Cheat Engine 7.7 macOS bundle:

| Metric | Value |
| --- | --- |
| File | `/Applications/CheatEngine77.app/Contents/Lua/celua.txt` |
| Lines | 4,444 |
| Size | ~240 KB |
| Unique function names | 585 |
| Signatures | 613 |

The difference between names and signatures is overloading. For example,
`openProcess` accepts a PID or a process name, and `loadTable` accepts a filename
or a stream.

An initial count of 586 included `ModuleLoader(Inheritance: -)`, a class heading
rather than a function. The corrected function count is 585.

The same document is published in the public source tree:
<https://github.com/cheat-engine/cheat-engine/blob/master/Cheat%20Engine/bin/celua.txt>

## How this project uses it

The API is the compatibility target for the Lua bridge.

- Function **names** are recorded in
  [`spec/lua-api.yaml`](../../spec/lua-api.yaml).
- Descriptions are **not copied**. The official document is the reference.
- Each name has a migration status: `unmapped`, `curated`, `generated`,
  `excluded` or `unsupported-on-macos`.
- The `lua` profile generates one MCP tool per name.
- Curated tools map to one or more functions.

This is an interoperability list, not a redistribution of Cheat Engine's
documentation.

## Main families

The following is a map of the API, not a full reference. Use `celua.txt` for
signatures and semantics.

### Process and modules

| Function | Purpose |
| --- | --- |
| `getProcesslist()` | List processes |
| `openProcess(pid)` / `openProcess(name)` | Open a process |
| `getOpenedProcessID()` | Current PID |
| `getProcessIDFromProcessName(name)` | Resolve a name to a PID |
| `enumModules()` | Enumerate modules |
| `getThreadlist()` | List thread IDs |

There is no documented `closeProcess`. `closeCE()` closes Cheat Engine, not the
target. The correct detach behavior is an open question.

### Memory

| Function | Purpose |
| --- | --- |
| `readBytes(address, count)` | Read bytes |
| `writeBytes(address, bytes)` | Write bytes |
| `readInteger` / `writeInteger` | 32-bit integer |
| `readQword` / `writeQword` | 64-bit integer |
| `readFloat` / `writeFloat` | 32-bit float |
| `readDouble` / `writeDouble` | 64-bit float |
| `readString` / `writeString` | String |
| `readPointer` / `writePointer` | Pointer-sized value |
| `allocateMemory(size, protection)` | Allocate in the target |
| `deAlloc(address)` | Free an allocation |
| `setMemoryProtection(address, size, rights)` | Change protection |
| `enumMemoryRegions()` | Enumerate regions |

### Scans

| Function | Purpose |
| --- | --- |
| `createMemScan(progressbar)` | Create a scan object |
| `memscan:firstScan(...)` | First scan |
| `memscan:nextScan(...)` | Next scan |
| `memscan:waitTillDone()` | Wait for completion |
| `memscan:getResultCount()` | Result count |
| `memscan:getResults(...)` | Read results |

The scan object exposes the usual Cheat Engine scan options: value type,
rounding, start/end address, protection flags, alignment, hexadecimal input,
Unicode and case sensitivity.

### Address list and tables

| Function | Purpose |
| --- | --- |
| `getAddressList()` | Address-list object |
| `addresslist.createMemoryRecord()` | Create a record |
| `loadTable(filename, merge)` | Load `.CT` / `.CETRAINER` |
| `saveTable(filename, protect)` | Save a table |
| `registerSymbol(name, address)` | Register a symbol |
| `getNameFromAddress(address)` | Address to symbol/module offset |

The address-list object exposes record properties, freeze state, hotkeys and
script activation.

### Assembly

| Function | Purpose |
| --- | --- |
| `assemble(line, address)` | Assemble one instruction |
| `disassemble(address)` | Disassemble one instruction |
| `getInstructionSize(address)` | Instruction length |
| `autoAssemble(script)` | Run an Auto Assembler script |
| `autoAssembleCheck(script)` | Syntax check |
| `AOBScan(pattern)` | Array-of-bytes scan |

### Debugger

| Function | Purpose |
| --- | --- |
| `debug_isDebugging()` | Debugger active |
| `debug_getCurrentDebuggerInterface()` | Windows, VEH, kernel, macOS native, gdb |
| `debug_setBreakpoint(address, ...)` | Set a breakpoint |
| `debug_removeBreakpoint(address)` | Remove a breakpoint |
| `debug_continueFromBreakpoint(method)` | Continue, step into or step over |
| `debug_getContext(extraregs)` | Read registers |
| `debug_setContext(extraregs)` | Write registers |
| `debug_getCurrentContextTable(extraregs)` | Context as a table |

The macOS native interface is value `4` in
`debug_getCurrentDebuggerInterface()`. arm64 context support is not verified.

### Structures

| Function | Purpose |
| --- | --- |
| `getStructureCount()` | Number of global structures |
| `getStructure(index)` / `getStructure(name)` | Get a structure |
| `registerStructureDissectOverride(...)` | Custom dissect behavior |
| `getStructureElementsFromName(name)` | PDB elements |

### Lua, threads and utilities

| Function | Purpose |
| --- | --- |
| `createThread(function)` | Create a Lua thread |
| `createThreadNewState(script)` | Thread with a separate Lua state |
| `synchronize(function)` | Run on the main thread |
| `queue(function)` | Queue on the main thread |
| `checkSynchronize(timeout)` | Process the queue |
| `createServerSocket(port, localhostOnly)` | Open a TCP listener |
| `acceptConnection()` | Accept a connection |
| `getMainForm()` / `getMemoryViewForm()` | Form objects |
| `getLuaEngine()` | Lua engine form |

The bridge depends on `createServerSocket`, `acceptConnection`, `createThread`
and `synchronize`. `acceptConnection` blocks its calling thread; the bridge must
run it in a Lua thread. `LuaPipe` provides `readBytes`, `writeBytes`, `Timeout`,
`OnTimeout` and `OnError` for the framed JSON payload.

## Platform notes

`celua.txt` is cross-platform, but not every function works everywhere.

- `dbvm_*` and `dbk_*` target Windows kernel/DBVM features. They are excluded
  from the macOS bridge.
- `debug_*` includes Windows-specific behavior. The macOS native interface is
  `4`; arm64 context support is uncertain.
- `trainer_generate`-style workflows are Windows/PE-centric and are out of scope
  for the first macOS release.
- File dialogs and form automation may behave differently on macOS.

The migration status per function is in `spec/lua-api.yaml`, not in this page.

## Related

- [Coverage](coverage.md)
- [Tool migration](../migration/06-tool-migration.md)
- [Target inventory](../migration/01-target-inventory.md)
- [Official celua.txt](https://github.com/cheat-engine/cheat-engine/blob/master/Cheat%20Engine/bin/celua.txt)
