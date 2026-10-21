# Migration overview

This section records how `ce-mcp` moved from "a full Cheat Engine clone in Rust"
to an MCP-first design with a Lua bridge, and what evidence supports each
decision.

The migration is documented as a sequence of dated notes, not as a polished
after-the-fact story. Where a claim comes from a command, the command and its
limits are recorded. Where a conclusion is an inference, it is labelled as
such.

## Timeline

| Date | Milestone |
| --- | --- |
| 2026-10-21 | Inspected the Cheat Engine 7.7 macOS bundle and measured its surface |
| 2026-10-21 | Researched the public Cheat Engine source landscape |
| 2026-10-21 | Researched MCP `2026-07-28` and the Rust SDK |
| 2026-10-21 | Narrowed the scope from a full clone to an MCP server |
| 2026-10-21 | Chose a Lua bridge as the first backend |
| 2026-10-21 | Defined profiles, stateless handles and the tool catalog |
| 2026-10-21 | Decided to publish a documentation-only public preview |
| TBD | F0: bridge and MCP spike |
| TBD | Implementation license selected before code contributions |

## How to read these documents

| Document | Question it answers |
| --- | --- |
| [01-target-inventory.md](01-target-inventory.md) | What exactly is the Cheat Engine 7.7 build we are targeting? |
| [02-source-landscape.md](02-source-landscape.md) | What public source exists, and what can we rely on? |
| [03-mcp-research.md](03-mcp-research.md) | Which MCP revision and transports should we target? |
| [04-decisions.md](04-decisions.md) | What did we decide, when, and why? |
| [05-backend-strategy.md](05-backend-strategy.md) | Why a Lua bridge first, and how does a native backend fit? |
| [06-tool-migration.md](06-tool-migration.md) | How do Cheat Engine capabilities become MCP tools? |
| [07-open-source.md](07-open-source.md) | What does the project need before it is open source? |
| [journal/2026-10-21.md](journal/2026-10-21.md) | What happened in the first design session? |

## Evidence labels

Every technical claim in this section should be read with one of these labels:

- **Observation** — directly measured from the shipped bundle, the public
  source tree or an official specification.
- **Inference** — a conclusion drawn from observations; may be wrong.
- **Unknown** — not established by the available evidence.

The initial inventory is based on shell tools (`file`, `nm`, `strings`,
`otool`, `codesign`), not on a formal REA analysis. This is called out in
[01-target-inventory.md](01-target-inventory.md).

## Scope of the migration

"Migration" here means two things:

1. **Capability migration:** turning Cheat Engine's operations into stable MCP
   tools.
2. **Backend migration:** moving from the Lua bridge to a native Rust backend
   without changing the MCP contract.

It does **not** mean porting the Cheat Engine GUI to Rust. The desktop UI is
out of scope for the current roadmap.

## Maintenance

When a new investigation or decision is made:

1. add a dated journal entry under `journal/`;
2. update [04-decisions.md](04-decisions.md);
3. add or supersede an ADR if the decision is architectural;
4. update [06-tool-migration.md](06-tool-migration.md) if the tool catalog
   changes;
5. keep the coverage document honest: designed is not implemented.
