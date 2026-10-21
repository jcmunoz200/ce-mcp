# ADR-0004: Expose tools through profiles

- **Status:** Accepted
- **Date:** 2026-10-21
- **Supersedes:** none
- **Superseded by:** none

## Context

Cheat Engine documents 585 unique Lua functions. Exposing all of them as
individual MCP tools by default would:

- consume a large part of the model's context with tool schemas;
- slow down tool selection and increase wrong-tool calls;
- make the common workflow harder to discover;
- destabilize prompt caching when the tool list changes.

At the same time, the project's goal is to expose **all** tools. Hiding the full
API would fail that goal.

## Decision

Use four explicit profiles:

| Profile | Contents | Default |
| --- | --- | --- |
| `core` | ~30 curated tools for process, memory, scan, table and Lua | Yes |
| `full` | All ~80 curated tools, including debugger, assembler and structures | No |
| `lua` | One generated tool per documented Lua function (~585) | No |
| `all` | `full` + `lua` | No |

Additional rules:

- The server advertises the active profile in `server/discover` and
  `ce_capabilities`.
- The profile is chosen by server configuration, not by a tool call.
- `tool_search` searches the full catalog and can describe tools outside the
  active profile; invoking one outside the profile returns
  `CE_TOOL_NOT_IN_PROFILE`.
- Tool names and schemas are stable across profiles; a tool present in `core` is
  identical in `full` and `all`.
- Generated Lua tools use a deterministic prefix and are marked as generated in
  their metadata.
- The `lua` profile is a compatibility surface, not the recommended default.

## Consequences

### Positive

- The default context stays small and focused.
- Power users can opt into the full curated surface.
- Full Lua compatibility remains available.
- Profiles make coverage and testing measurable.
- Tool lists are deterministic and cacheable.

### Negative

- Users must understand profiles and configure the server.
- A tool can be "available" but outside the active profile, which can confuse
  clients.
- Four profiles multiply the test matrix.
- The generated tools still add a large schema set when enabled.

### Neutral

- `lua_eval` is always available in `core` and `full`, so a user is never
  completely blocked by a missing curated tool.
- `tool_search` mitigates discoverability without loading every schema.

## Alternatives considered

1. **Expose all 585 tools by default.** Rejected: context and selection cost.
2. **One generic `ce_call` tool.** Rejected: no typed schemas, poor
   discoverability and weak validation.
3. **Dynamically add tools after a search.** Rejected: tool lists become
   unstable and incompatible with client-side caching.
4. **Separate server binaries per profile.** Rejected: duplicated deployment and
   confusing identity.

## References

- [Profiles](../mcp/profiles.md)
- [Tool catalog](../mcp/tool-catalog.md)
- [Coverage](../reference/coverage.md)
