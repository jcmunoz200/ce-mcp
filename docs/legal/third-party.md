# Third-party notices

This repository references third-party projects. It does not vendor them unless
stated otherwise. The table is informational and not legal advice.

| Project | Use | License | Link |
| --- | --- | --- | --- |
| Cheat Engine | Target application; not distributed | See upstream | <https://cheatengine.org/> |
| Model Context Protocol | Protocol specification | See MCP project | <https://modelcontextprotocol.io/> |
| `rmcp` (Rust SDK) | Planned MCP implementation dependency | Apache-2.0, with some MIT-licensed contributions | <https://github.com/modelcontextprotocol/rust-sdk> |
| mdBook | Documentation site generator | MIT | <https://github.com/rust-lang/mdBook> |
| markdownlint-cli2 | Documentation linting | MIT | <https://github.com/DavidAnson/markdownlint-cli2> |
| lychee | Link checking | MIT or Apache-2.0 | <https://github.com/lycheeverse/lychee> |
| Contributor Covenant | Code of conduct | CC BY 4.0 | <https://www.contributor-covenant.org/> |
| `rxi/json.lua` | Planned Lua JSON helper; not vendored yet | MIT | <https://github.com/rxi/json.lua> |
| GitHub Actions | CI | GitHub terms | <https://github.com/features/actions> |

## Cheat Engine

Cheat Engine is a separate application with its own license and terms. This
repository does not distribute it and does not grant any rights to it.

The project records factual API function names from `celua.txt` for
interoperability. It does not reproduce the documentation's descriptions. The
official document is linked from
[the Lua API reference](../reference/ce-lua-api.md).

## MCP specification

The MCP specification is maintained by the Model Context Protocol project. The
project's licensing is in transition; see the
[rust-sdk LICENSE](https://github.com/modelcontextprotocol/rust-sdk/blob/main/LICENSE)
and the [MCP specification repository](https://github.com/modelcontextprotocol/modelcontextprotocol).

## rmcp

`rmcp` is the official Rust SDK for MCP. Its LICENSE describes a transition from
the MIT License to Apache-2.0:

- new code and specification contributions are Apache-2.0;
- documentation contributions are CC BY 4.0;
- some earlier contributions remain MIT.

If `ce-mcp` depends on `rmcp`, the final implementation license must be
compatible with these terms. This will be checked before F0.

## mdBook

The documentation site is built with mdBook, which is MIT-licensed. The
repository contains only configuration and Markdown; it does not vendor mdBook.

## markdownlint and lychee

CI uses markdownlint-cli2 and lychee as GitHub Actions. They are not part of the
shipped software.

## Contributor Covenant

The code of conduct links to and adapts the Contributor Covenant 2.1, which is
licensed under CC BY 4.0. Attribution is included in
[CODE_OF_CONDUCT.md](../../CODE_OF_CONDUCT.md).

## json.lua

The bridge may vendor `rxi/json.lua` for JSON encoding inside Cheat Engine. If
it does, the repository will include:

- the MIT license text;
- the upstream commit or version;
- a hash of the vendored file;
- a note in this document.

It is not vendored in the current documentation-only preview.

## No Cheat Engine assets

No Cheat Engine binary, source file, language file, tutorial, icon, screenshot
or documentation text is included in this repository.

## Reporting

If you believe a third-party notice is missing or inaccurate, open an issue or
contact the maintainer. See [MAINTAINERS.md](../../MAINTAINERS.md).
