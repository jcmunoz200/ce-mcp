# Security policy

## Supported versions

There is no implementation yet. This policy covers the design and the planned
implementation.

## Reporting a vulnerability

Please report security issues privately through GitHub Security Advisories for
this repository. Do not open a public issue.

Include:

- the affected component (MCP server, bridge, documentation);
- a description of the issue and its impact;
- reproduction steps or a proof of concept;
- the version or commit you tested.

There is no bug bounty.

## Security model

`ce-mcp` is a local development tool. Its security model is deliberately
similar to Cheat Engine's:

- The bridge listens on localhost only. It must never bind to `0.0.0.0` or a
  public interface by default.
- The bridge authenticates with a per-launch token stored in a user-only file.
- The bridge executes Lua in Cheat Engine. That is arbitrary code execution by
  design; only load bridge scripts you trust.
- Once a process is opened, memory writes are allowed. This matches Cheat
  Engine and is not a vulnerability.
- The MCP server does not bypass SIP, code signing, hardened runtime or
  anti-cheat protections.
- The project does not ship kernel drivers or DBVM.

## Out of scope

- Vulnerabilities in Cheat Engine itself: report them to the Cheat Engine
  authors.
- Anti-cheat or game-protection bypass.
- Attacks that require the user to run an untrusted bridge script.
- Issues caused by exposing the bridge to a non-local network.

## Planned hardening

- Token file permissions `0600`.
- Bind to `127.0.0.1` and verify the peer address.
- Reject unauthenticated requests before evaluating Lua.
- Optional audit log of tool calls.
- Resource limits for reads, scan results and Lua execution time.
- No remote transport enabled by default.
