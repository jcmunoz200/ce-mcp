# Security

## Scope

`ce-mcp` is a local development and research tool. It gives an MCP client the
same power a Cheat Engine user has: read and write another process's memory,
patch code, run Lua and attach a debugger.

That power is the product. The design does not pretend to sandbox it.

## Threat model

| Threat | Impact | Control |
| --- | --- | --- |
| Another local process connects to the bridge | Arbitrary memory access and Lua execution | Localhost bind, per-launch token, `0600` state file |
| Bridge exposed on a public interface | Remote code execution in Cheat Engine's context | Loopback-only default; explicit `--allow-remote` with warning |
| Malicious bridge script | Full control of Cheat Engine | Bridge is opt-in; users must review scripts |
| Prompt-injected agent writes to the wrong process | Data corruption or crash | Process handle is explicit; `process_info` before writes; optional audit |
| Agent reads sensitive memory | Data exposure to the model/provider | Local tool only; user controls the target; no telemetry by default |
| Token leaked in logs or errors | Unauthorized bridge access | Redaction; token never included in tool results |
| Dependency compromise | Code execution in the MCP server | Minimal dependencies, lockfile, dependency review |
| Privilege escalation | Access to protected processes | The server never bypasses SIP, code signing or OS permissions |

## Bridge security

The bridge is a Lua script running inside Cheat Engine.

- **Bind:** `127.0.0.1` only, using `createServerSocket(port, true)`.
- **Port:** chosen at startup and written to the state file. No fixed port.
- **Token:** random per launch, at least 128 bits, required on every request
  before any Lua is evaluated.
- **State file:** `~/.ce-mcp/bridge.json`, mode `0600`, contains port, token, PID
  and protocol version.
- **Authentication:** constant-time comparison where feasible; reject and close
  on mismatch.
- **No unauthenticated eval:** the `eval` and `call` operations require a valid
  token. `ping` may be unauthenticated but returns no sensitive data.
- **Limits:** maximum request size, maximum response size, execution deadline,
  maximum concurrent connections.
- **Shutdown:** a `shutdown` request closes the listener and removes the state
  file.

The bridge does not use TLS. Loopback traffic does not leave the host, and the
token protects against other local users. A future remote mode would require
TLS and a different trust model.

## Arbitrary Lua

`lua_eval`, `lua_call` and the `lua` profile execute arbitrary code inside Cheat
Engine. This is intentional.

- There is no Lua sandbox.
- A malicious script can read files, open sockets, modify the target and crash
  Cheat Engine.
- Only run scripts you trust.
- The MCP server does not automatically run scripts from a `.CT` file or from a
  model response without an explicit tool call.

If a future release adds a sandbox, it will be a separate backend with a
documented reduced API, not a silent change to `lua_eval`.

## Cheat Engine semantics

Once a process is opened, writes are allowed, matching Cheat Engine. See
[ADR-0005](../decisions/ADR-0005-ce-semantics.md).

This means:

- `process_open` is the consent boundary;
- an agent should call `process_info` before the first write;
- the optional `--safe` mode can reject `write` and `execute` tools, including
  `lua_eval`; it is not a sandbox and does not make an allowed tool safe;
- the optional audit log records calls but does not prevent them.

## macOS permissions

Cheat Engine uses `task_for_pid`, Mach VM APIs and, for debugging, Mach
exceptions or `ptrace`. macOS may require:

- root or a target signed with `get-task-allow`;
- disabling SIP for some operations;
- approving the app in System Settings → Privacy & Security;
- Developer Tools permissions for the terminal or client.

`ce-mcp` does not bypass these requirements. If an operation is denied, the tool
returns `CE_ACCESS_DENIED` and explains the OS-level remediation. It does not
attempt to disable SIP or escalate privileges.

## Data handling

- No telemetry, analytics or cloud calls by default.
- Memory reads are returned to the MCP client; the client and model provider may
  store them.
- The server does not persist memory dumps unless a tool explicitly writes a
  file.
- Audit logs are opt-in and redact tokens.
- The bridge token is never included in tool results, logs or error causes.

## HTTP mode

If Streamable HTTP is enabled:

- bind to `127.0.0.1` by default;
- require a token;
- validate the `Origin` header for browser clients;
- do not enable CORS wildcards;
- do not expose the endpoint through a reverse proxy without authentication;
- prefer stdio when a single local client is enough.

The HTTP server is not a multi-tenant service. It is a local control plane.

## Supply chain

- The Rust implementation will use a lockfile and minimal dependencies.
- `rmcp` is the only protocol dependency planned for the first release.
- The Lua bridge will avoid vendored third-party Lua modules where possible.
- If `json.lua` is vendored, its MIT license and hash will be recorded in
  [Third-party notices](../legal/third-party.md).
- CI will run dependency review and secret scanning.

## What is out of scope

- Anti-cheat or game-protection bypass.
- Kernel drivers, DBVM and Windows kernel features.
- Hiding Cheat Engine or the MCP server from other processes.
- Protecting a target from its own user.
- Sandboxing untrusted Lua.

## Reporting

Report vulnerabilities privately through GitHub Security Advisories. See
[SECURITY.md](../../SECURITY.md). Do not open a public issue for a security
problem.

## References

- [ADR-0005](../decisions/ADR-0005-ce-semantics.md)
- [ADR-0006](../decisions/ADR-0006-lua-bridge.md)
- [Errors](errors.md)
- [CWE-306: Missing Authentication for Critical Function](https://cwe.mitre.org/data/definitions/306.html)
