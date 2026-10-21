# ce-mcp

**Unofficial MCP server for Cheat Engine — design preview.**

This book is the design and migration documentation for `ce-mcp`. The project
does not have an implementation yet.

> **License:** no license is granted at this time. This is a public design
> preview, not an open-source project yet. See
> [License status](legal/license-status.md).

## Start here

- [Vision](vision.md) — what the project is for and what it is not.
- [Architecture](architecture.md) — MCP layer, backend contract and bridge.
- [Migration overview](migration/README.md) — how the design evolved.
- [Target inventory](migration/01-target-inventory.md) — what was found in
  Cheat Engine 7.7.
- [MCP research](migration/03-mcp-research.md) — the `2026-07-28` protocol.
- [Tool catalog](mcp/tool-catalog.md) — planned tools and profiles.
- [Coverage](reference/coverage.md) — what is designed, mapped or missing.

## Status at a glance

| Area | Status |
| --- | --- |
| Target inventory | Complete for the shipped Cheat Engine 7.7 macOS bundle |
| MCP protocol research | Complete for `2026-07-28` |
| Tool catalog | Designed; no implementation |
| Lua bridge | Designed; not implemented |
| MCP server | Designed; not implemented |
| Documentation site | This book |
| License | TBD; public preview |

## Scope

The first deliverable is an MCP server that drives Cheat Engine through a local
Lua bridge. A native Rust backend may follow behind the same contract. The
project is not a Cheat Engine clone and does not redistribute Cheat Engine.

See [Vision](vision.md) for the full scope and non-goals.
