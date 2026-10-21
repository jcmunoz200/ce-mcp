# ADR-0001: Adopt an MCP-first architecture

- **Status:** Accepted
- **Date:** 2026-10-21
- **Supersedes:** the full-clone plan
- **Superseded by:** none

## Context

The original goal was a complete Rust clone of Cheat Engine 7.7, including its
GUI, scanner, debugger, assembler, Lua runtime and trainer generation.

The target inventory showed the size of that plan:

- an 83,482,352-byte main binary;
- 119,479 FPC-mangled symbols in the arm64 slice;
- 136 `Tfrm*` classes and 344 `TPF0` resources;
- 585 unique documented Lua functions;
- a bundled Lua 5.3 runtime, TCC, speedhack and two tutorial applications.

A faithful clone would take years before it delivered a usable interface. In
parallel, Cheat Engine already exposes most of its capability through Lua. The
missing piece is a standard interface for agents, not another engine.

## Decision

The product is an MCP server, `ce-mcp`.

- The MCP contract is the stable surface.
- The first backend is a Lua bridge into Cheat Engine 7.7.
- A native Rust backend may replace the bridge later behind the same contract.
- The desktop GUI, full native engine and trainer generation are deferred.

## Consequences

### Positive

- The MCP design can be validated against a real application quickly.
- Cheat Engine semantics are preserved by construction.
- The documented Lua API provides broad initial coverage.
- The project has a clear, reviewable scope.
- Native work can proceed incrementally without breaking clients.

### Negative

- The first backend requires a Cheat Engine installation.
- macOS permissions and SIP constraints still apply.
- Some features have no reliable Lua API and remain gaps.
- The project depends on Cheat Engine's Lua behavior, including its bugs.

### Neutral

- A future native backend is still possible; it is no longer a prerequisite.
- The repository name `ce-mcp` reflects the narrowed scope.

## Alternatives considered

1. **Full Rust clone first.** Rejected: too large, delays validation, and
   duplicates a working tool.
2. **Library-only Rust wrapper.** Rejected: does not give agents a standard
   discovery and invocation surface.
3. **MCP server that automates the GUI.** Rejected: fragile, inaccessible and
   hard to test.
4. **MCP server with a native engine from day one.** Rejected for the current
   phase: the contract would be designed against an incomplete engine.

## References

- [Migration overview](../migration/README.md)
- [Tool migration](../migration/06-tool-migration.md)
- [ADR-0006](ADR-0006-lua-bridge.md)
