# ADR-0005: Preserve Cheat Engine write semantics

- **Status:** Accepted
- **Date:** 2026-10-21
- **Supersedes:** none
- **Superseded by:** none

## Context

Many agent tools default to read-only or require per-operation confirmation.
Cheat Engine does not. Once a process is opened, Cheat Engine allows reading and
writing memory, changing protections, patching code and running scripts. That is
the behavior users expect.

The project could add a safety layer, but a different default would:

- break common Cheat Engine workflows;
- make tools behave differently from the application they wrap;
- create a false sense of safety, because `lua_eval` can write anyway;
- add friction to every experiment.

## Decision

`ce-mcp` preserves Cheat Engine semantics.

- `process_open` is the consent boundary. After it succeeds, memory writes,
  protection changes and code patches are allowed.
- There is no read-only mode by default.
- Tools declare a risk class (`read`, `write`, `execute`, `debug`, `ui`) for
  documentation and optional policy.
- An opt-in `--safe` mode may reject `write` and `execute` tools. It is not the
  default and does not sandbox Lua.
- An opt-in audit log records tool calls, arguments and results. It is off by
  default.
- The project never claims to make Cheat Engine safe. It documents the risks.

## Consequences

### Positive

- Behavior matches Cheat Engine.
- Existing workflows and scripts transfer directly.
- No surprising failures when an agent tries to edit a value.
- The risk model is honest: `lua_eval` is arbitrary execution by design.

### Negative

- An agent can corrupt a target process.
- A prompt-injected agent could write to the wrong process.
- Users may expect the tool to protect them and be surprised.
- The optional safe mode can create a false sense of security.

### Neutral

- The audit log is an observability feature, not a security boundary.
- Risk classes are metadata; clients may use them for their own approvals.

## Alternatives considered

1. **Read-only by default.** Rejected: diverges from Cheat Engine, blocks the
   main use case and is bypassable through Lua.
2. **Confirm every write.** Rejected: there is no reliable GUI confirmation
   channel in a headless MCP server; it would break automation.
3. **No write tools at all.** Rejected: fails the product goal.
4. **Sandbox Lua.** Rejected: Cheat Engine's API is the product; a sandbox would
   break compatibility and still leave memory writes.

## References

- [Security](../mcp/security.md)
- [Errors](../mcp/errors.md)
- [Tool catalog](../mcp/tool-catalog.md)
