# Decision log

This is the human-readable index of decisions. Architectural decisions have a
full ADR in [`docs/decisions/`](../decisions/).

| ID | Date | Decision | Rationale | Status |
| --- | --- | --- | --- | --- |
| D1 | 2026-10-21 | Make MCP the product, not an afterthought | Cheat Engine already exposes a large Lua API; an MCP server delivers value without reimplementing the GUI | Accepted, [ADR-0001](../decisions/ADR-0001-mcp-first.md) |
| D2 | 2026-10-21 | Scope the current project to the MCP server | A full Rust clone is a multi-year effort and would delay validation of the MCP design | Accepted, [ADR-0001](../decisions/ADR-0001-mcp-first.md) |
| D3 | 2026-10-21 | Use a Lua bridge as the first backend | Immediate coverage of the documented API and exact Cheat Engine semantics | Accepted, [ADR-0006](../decisions/ADR-0006-lua-bridge.md) |
| D4 | 2026-10-21 | Preserve Cheat Engine write semantics | Once a process is open, Cheat Engine allows writes; adding a different default would break compatibility | Accepted, [ADR-0005](../decisions/ADR-0005-ce-semantics.md) |
| D5 | 2026-10-21 | Use `core`, `full`, `lua` and `all` tool profiles | Hundreds of tools in every context would hurt discovery and selection | Accepted, [ADR-0004](../decisions/ADR-0004-tool-profiles.md) |
| D6 | 2026-10-21 | Default to stdio; offer Streamable HTTP on localhost | stdio is the lowest-overhead local transport; HTTP is needed for multi-client and embedded mode | Accepted, [ADR-0003](../decisions/ADR-0003-transports.md) |
| D7 | 2026-10-21 | Use stateless MCP with explicit handles | MCP `2026-07-28` has no sessions; state belongs in the backend | Accepted, [ADR-0002](../decisions/ADR-0002-stateless-handles.md) |
| D8 | 2026-10-21 | Publish a documentation-only public preview now | Makes the design reviewable and the migration reusable without publishing unfinished code | Accepted, [ADR-0008](../decisions/ADR-0008-public-preview.md) |
| D9 | 2026-10-21 | Write repository documentation in English | Standard for open-source reach and contributions | Accepted |
| D10 | 2026-10-21 | Do not choose a license yet | The code license decision can wait until implementation; the repository is explicitly marked as a public preview | Accepted, [ADR-0007](../decisions/ADR-0007-docs-license.md) |
| D11 | 2026-10-21 | Use REA CLI + Mach-O inventory first; add Ghidra only if needed | The public source covers most of the architecture; REA should target macOS-specific deltas | Accepted, pending execution |
| D12 | 2026-10-21 | Keep `.CT` and Cheat Engine Lua compatibility | Existing tables and scripts are the main ecosystem; a clean format would lose that value | Accepted |
| D13 | 2026-10-21 | Defer the egui desktop GUI | The MCP server and bridge must be validated first; the GUI is a separate client of the same engine | Superseded by D2 for the current scope |
| D14 | 2026-10-21 | Name the public repository `ce-mcp` | Short, neutral and not implying affiliation with Cheat Engine | Accepted |

## Decisions that were considered and deferred

| Topic | Why it is deferred |
| --- | --- |
| Full Rust engine | Requires reimplementing memory access, scanning, debugger, assembler and the Lua API before the MCP contract is validated |
| In-app desktop UI | A GUI is a large surface and not needed to prove the MCP design |
| Pointer scan | No obvious Lua API; needs either form automation or a native implementation |
| Windows/Linux bridge | The first target is macOS 7.7; the contract should stabilize first |
| `.CETRAINER` and trainer generation | Depends on a working table and script layer |
| Multi-agent HTTP server | F1/F4 work; stdio is enough for F0 |

## How decisions are changed

Open a Design proposal issue. If the change is architectural, add a new ADR that
supersedes the old one and update this table. Do not rewrite history: a
superseded ADR stays in the repository.

## Open decisions

- The implementation license.
- The exact bridge wire format after the F0 spike.
- Whether the pointer scan gap is filled natively or through the existing
  Cheat Engine forms.
- Whether the first public implementation release supports MCP `2025-11-25`
  clients.
