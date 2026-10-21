# Glossary

## Cheat Engine terms

**Address list**
: The table of memory records in Cheat Engine. Each record has an address, a
  value type, a description, an optional script and a freeze state.

**AOB (array of bytes)**
: A byte pattern, often with wildcards, used to find code or data that moves
  between runs.

**Auto Assembler (AA)**
: Cheat Engine's script language for allocating memory, injecting code,
  registering symbols and enabling/disabling patches.

**Cheat table (`.CT`)**
: A Cheat Engine table file containing address-list records, scripts, symbols
  and forms.

**`.CETRAINER`**
: A protected table format used for trainers. Support is Windows-centric and is
  not a first-release goal.

**Code cave**
: Unused space in a module or memory region where injected code can be placed.

**DBVM**
: Cheat Engine's hypervisor-based kernel access layer on Windows. Out of scope
  for macOS.

**Freeze**
: Continuously write a value to keep it from changing.

**Hotkey**
: A key combination that triggers a table entry or action.

**MemScan**
: The Cheat Engine scan object returned by `createMemScan`. It holds the scan
  type, value type, range and results.

**Memory record**
: An entry in the address list. It can point to an address, a pointer path or a
  script.

**Pointer map**
: A snapshot of pointers in a process, used to accelerate pointer scans.

**Pointer scan**
: A search for a chain of pointers that reaches a target address. Cheat Engine's
  implementation is form-driven; the Lua API does not expose a direct scan call.

**Speedhack**
: A Cheat Engine feature that changes the perceived speed of a target.

**Structure dissect**
: A Cheat Engine view that interprets a memory region as a structure with typed
  fields.

**Symbol**
: A name mapped to an address, such as `game.exe+0x1234` or a user-defined
  symbol.

**TCC**
: Tiny C Compiler. Cheat Engine bundles it for compiled scripts.

**VEH**
: Vectored Exception Handling, a Windows debugger mechanism. Out of scope for
  macOS.

## MCP terms

**Capability**
: A feature a client or server supports, advertised during discovery.

**Cursor**
: An opaque token for pagination.

**Handle**
: An opaque server-side reference to state, such as a process or scan. Handles
  are used because MCP is stateless.

**MCP**
: Model Context Protocol, a JSON-RPC protocol for exposing tools, resources and
  prompts to AI clients.

**`server/discover`**
: The `2026-07-28` method that returns supported protocol versions, capabilities
  and server identity.

**SSE**
: Server-Sent Events. In Streamable HTTP, a request-scoped stream for progress
  and multi-message responses.

**Stateless**
: No session or initialization handshake. Every request carries its own
  metadata.

**stdio**
: Newline-delimited JSON-RPC over a client-launched subprocess's standard
  streams.

**Streamable HTTP**
: The standard HTTP transport where each message is a POST and the response is
  JSON or a request-scoped SSE stream.

**Task**
: An extension for long-running work. The server returns a task ID; the client
  polls `tasks/get` and can cancel with `tasks/cancel`.

**`ttlMs` / `cacheScope`**
: Cache hints on list and resource results. `ttlMs` is a freshness duration;
  `cacheScope` is `public` or `private`.

## Project terms

**Backend**
: The component that executes operations. The first backend is the Lua bridge;
  a native Rust backend may follow.

**Bridge**
: `ce_bridge.lua`, the script loaded into Cheat Engine that exposes a local
  socket and evaluates requests.

**Curated tool**
: A typed, documented MCP tool with a stable contract, as opposed to a generated
  Lua tool.

**Generated tool**
: An MCP tool created automatically from a documented Lua function, available
  under the `lua` profile.

**Profile**
: A named set of tools: `core`, `full`, `lua` or `all`.

**REA**
: Reverse Engineering Agent, the tool-assisted analysis workflow used to inspect
  the Cheat Engine binary. The initial inventory was shell-based; a formal REA
  pass is pending.

**Safe mode**
: An optional mode that rejects write and execute tools. It is not a sandbox and
  is not the default.

## Related

- [Architecture](../architecture.md)
- [Protocol](../mcp/protocol.md)
- [Tool catalog](../mcp/tool-catalog.md)
