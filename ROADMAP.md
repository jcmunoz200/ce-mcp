# Roadmap

The project is currently in **D0: design and migration documentation**. There is
no implementation yet.

## D0 — Public design preview

Status: complete for the documentation scope. The public preview is at
<https://jcmunoz200.github.io/ce-mcp/>. Implementation starts in F0 after the
license is chosen.

- [x] Initial target inventory of Cheat Engine 7.7.
- [x] Research the MCP `2026-07-28` specification.
- [x] Decide MCP-first scope and Lua-bridge backend.
- [x] Define the tool catalog and profiles.
- [x] Run a formal REA CLI/Mach-O, plist and signature pass.
- [x] Publish the documentation site.
- [x] Review every claim and link with markdownlint, lychee and mdBook.
- [ ] Close the open design questions in the migration journal; these are
  tracked for F0 and F3.

Exit criteria:

- a public, readable repository that explains the migration and the MCP design;
- a machine-readable tool catalog in `spec/`;
- an explicit license status and contribution policy;
- no Cheat Engine assets or copied documentation.

## F0 — Bridge and MCP spike

Status: planned.

- Implement `bridge/ce_bridge.lua` with localhost-only TCP, token auth and
  length-prefixed JSON.
- Implement `ce-mcp` with `rmcp` 3.6 over stdio.
- Expose six tools: `ce_info`, `process_list`, `process_open`, `memory_read`,
  `scan_start` and `scan_results`.
- Validate against a target process owned by the tester.
- Choose the license for the implementation **before** merging code.
- Publish the first alpha.

Exit criteria: an MCP client can open a process, read memory and run a first
scan against Cheat Engine 7.7.

## F1 — Core profile

Status: planned.

- ~30 tools covering process, memory, scan, address list, `.CT` load/save and
  `lua_eval`.
- Streamable HTTP transport for the embedded/desktop mode.
- Stateless handles with TTL and explicit close.
- Task-based long scans and progress notifications.
- Optional audit log.

Exit criteria: a complete find → filter → edit → save workflow driven entirely
through MCP.

## F2 — Full profile and generated Lua tools

Status: planned.

- ~80 curated tools.
- `tool_search` and `ce_capabilities`.
- Generated tools for the documented Lua API under the `lua` profile.
- Documentation site with generated tool reference.
- Coverage checks in CI.

Exit criteria: every documented Lua function is either mapped to a tool,
generated as a tool, or explicitly excluded with a reason.

## F3 — Native gaps

Status: planned.

- Implement tools that have no reliable Lua API, starting with pointer scan and
  structure dissect.
- Add a native Rust backend behind the same contract.
- Parity tests against the Lua bridge.

Exit criteria: the declared coverage matrix has no unexplained gaps.

## F4 — Stable release

Status: planned.

- Multi-client Streamable HTTP, OpenTelemetry traces and metrics.
- Windows and Linux bridge variants.
- Stable `v1.0` with a documented compatibility policy.

## Out of scope

- A full Cheat Engine clone or GUI.
- Cheat Engine redistribution.
- Anti-cheat bypass, kernel drivers or DBVM.
- A read-only default that diverges from Cheat Engine semantics.

See [docs/migration/04-decisions.md](docs/migration/04-decisions.md) for the
reasoning behind these boundaries.
