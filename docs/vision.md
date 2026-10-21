# Vision

## The problem

Cheat Engine is a powerful environment for inspecting and modifying processes,
but almost all of its capability is exposed through a desktop GUI. Automation
exists through Lua, yet an AI agent cannot discover or use it without a bespoke
integration for every workflow.

The Model Context Protocol gives agents a standard way to discover and call
tools. What is missing is a well-designed MCP surface for Cheat Engine's
capabilities.

## The idea

`ce-mcp` exposes Cheat Engine to MCP clients with three layers:

1. **Curated tools** for the common workflows: process selection, memory access,
   scans, the address list, symbols, the debugger, assembly and structures.
2. **A Lua escape hatch** for the full documented API.
3. **Generated tools** for every documented Lua function, behind an optional
   profile.

The MCP contract is the product. The first backend is a local Lua bridge into
Cheat Engine 7.7. A native Rust backend can replace it later without changing
tool names, parameters or semantics.

## Who it is for

- People who already use Cheat Engine and want an agent to handle repetitive
  workflows.
- Reverse-engineering researchers who want scriptable, auditable access to
  Cheat Engine's tools.
- MCP client and agent developers who need a realistic, stateful tool surface
  to test against.

## Principles

### Capability parity, not UI parity

The goal is every operation, not every dialog. A settings dialog becomes
parameters on the operation it configures. The project does not clone the
Cheat Engine GUI.

### Cheat Engine semantics

Once a process is opened, reads and writes are allowed, as they are in Cheat
Engine. The project does not add a read-only mode by default and does not
pretend to be safer than the underlying tool. It does document the risks and
offers optional auditing.

### Local first

The bridge listens on localhost only and authenticates with a per-launch token.
There is no cloud service, no remote execution and no telemetry by default.

### Spec first

The tool catalog is machine-readable. Documentation, schemas and tests are
generated from the same source of truth. Coverage is measured, not claimed.

### Backend independence

Tools call a backend contract, not Mach APIs or Lua directly. The Lua bridge is
the first implementation; a native backend is a later, independent milestone.

## Non-goals

- Reimplementing the Cheat Engine GUI.
- Redistributing Cheat Engine or its assets.
- Anti-cheat or game-protection bypass.
- Kernel drivers, DBVM or Windows-only kernel features.
- A read-only mode that changes Cheat Engine semantics by default.
- A cloud service that executes tools on someone else's machine.

## Success criteria

The project succeeds when an MCP client can:

- discover the available tools and their profiles;
- open a process, enumerate modules and regions, and read memory;
- run a first scan and refine it with the common scan types;
- edit values and manage address-list entries and `.CT` files;
- run Lua and Auto Assembler scripts;
- set breakpoints and inspect registers and stack traces;
- understand exactly which documented Cheat Engine functions are covered,
  generated or intentionally excluded.

The coverage matrix is the contract. "All tools" means all documented
capabilities are accounted for, not that every internal dialog has a tool.
