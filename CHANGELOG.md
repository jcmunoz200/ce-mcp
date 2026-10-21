# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
The project does not have a released version yet.

## [Unreleased]

### Added

- Initial target inventory of Cheat Engine 7.7 for macOS.
- Research notes for MCP `2026-07-28`: stateless requests, `server/discover`,
  Streamable HTTP, tasks and caching.
- Migration documentation for the Lua-bridge-first backend strategy.
- Tool catalog design covering curated tools and generated Lua tools.
- Architecture decision records for MCP-first scope, stateless handles,
  transports, profiles, Cheat Engine semantics, the Lua bridge, the docs
  license status and the public preview.
- Open-source readiness documentation, security policy, governance and
  contribution policy.
- mdBook documentation site and documentation CI.
- Published the public documentation site at
  <https://jcmunoz200.github.io/ce-mcp/>.
- Formal REA 6.7.0 artifact, Mach-O, plist and code-signature evidence for the
  shipped Cheat Engine 7.7 bundle, including the macOS version-floor and
  `libspeedhack` architecture findings.

### Changed

- Scoped the project from a full Cheat Engine clone to an MCP server with a
  pluggable backend.
- Reclassified `debug_attach`, `debug_detach` and `debug_step` from `gap` to
  `mapped` after verifying `debugProcess`, `detachIfPossible` and
  `debug_continueFromBreakpoint` in `celua.txt`. The curated coverage is now
  70 mapped and 11 gaps.
- Added formal dispositions and closure criteria for the 11 remaining gaps,
  including the pointer scanner, stack traces, form opening and snapshot
  restore.
- Added an acceptance test plan for the six-tool F0 bridge and MCP spike.

### Fixed

- Nothing yet.

### Security

- Documented the localhost-only bridge, token authentication and arbitrary-Lua
  execution model in `docs/mcp/security.md` and `SECURITY.md`.
