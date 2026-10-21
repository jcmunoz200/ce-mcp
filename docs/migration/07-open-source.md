# Open-source readiness

## Current status

This repository is a **public design preview**, not an open-source project yet.

- There is no implementation.
- There is no license for the code.
- The documentation is published so the design can be reviewed and the
  migration can be reused as a reference.

Publishing a repository publicly is not the same as open source. Without a
license, default copyright applies: people can view the repository, but they do
not receive the right to copy, modify or redistribute it.

## Why the license is deferred

The implementation is not written yet. Choosing a license now would force a
decision before the project knows whether it will include code derived from
Cheat Engine, third-party Lua libraries or other components.

The project therefore separates two decisions:

1. **Publish the design now.** This is allowed, with an explicit "no license"
   notice.
2. **Choose the implementation license before code contributions.** This is a
   hard gate for F0.

The cost of deferring is that the public preview cannot be described as open
source and code contributions cannot be accepted. That is intentional and
documented.

## What must happen before code

Before the first code pull request is merged:

- [ ] Choose the implementation license.
- [ ] Add the full license text and update `LICENSE.md`.
- [ ] Update `README.md`, `Cargo.toml` and package metadata.
- [ ] Update `CONTRIBUTING.md` with the contribution and sign-off rules.
- [ ] Record the decision in an ADR that supersedes
      [ADR-0007](../decisions/ADR-0007-docs-license.md).
- [ ] Confirm that no Cheat Engine source or assets are included.
- [ ] Confirm third-party dependency licenses are compatible.
- [ ] Enable code scanning and dependency review in CI.

Until then, code pull requests are closed with a link to this document.

## Trademark and assets

- The project is unofficial and not affiliated with Cheat Engine.
- The name is used only to describe interoperability.
- No Cheat Engine logo, icon, screenshot, language file, tutorial or binary is
  included.
- Documentation links to the official project instead of copying its prose.
- A `NOTICE` file records the trademark disclaimer.

See [Trademark](../legal/trademark.md).

## Contribution policy

During the design preview:

- issues and design proposals are welcome;
- documentation corrections are welcome as issues;
- code pull requests are paused;
- security reports go through GitHub Security Advisories.

After the license is chosen, the project will use a lightweight sign-off
process (DCO or equivalent) rather than a CLA, unless the chosen license
requires otherwise.

## Documentation infrastructure

- Markdown sources in `docs/`.
- mdBook build for the documentation site.
- GitHub Pages deployment from the default branch.
- `markdownlint` for structure and consistency.
- Link checking for external references.
- No generated code or copied documentation in the site.

## Release plan

| Release | Contents | Gate |
| --- | --- | --- |
| `v0.0.0-docs` | Design preview | This document set is complete and reviewed |
| `v0.1.0-alpha` | Bridge + MCP spike | License chosen; F0 acceptance criteria met |
| `v0.2.0-alpha` | Core profile | F1 acceptance criteria met |
| `v1.0.0` | Stable contract | F4 acceptance criteria met |

Documentation releases do not imply an implementation license.

## Sustainability

- Keep the scope narrow: MCP server, bridge and documented native gaps.
- Prefer generated documentation and coverage over manual claims.
- Keep the bridge and backend contract small enough for one maintainer to
  review.
- Do not accept code contributions that cannot be maintained.
- Revisit governance when there is more than one active maintainer.

## Checklist for making the repository public

- [x] README with status, scope and license notice.
- [x] Migration documentation and decision log.
- [x] Architecture and tool catalog.
- [x] Security policy and code of conduct.
- [x] Governance and maintainer list.
- [x] Documentation CI.
- [ ] Review all external links.
- [ ] Confirm no Cheat Engine assets or copied prose.
- [ ] Publish the repository under the `ce-mcp` name.
- [ ] Enable GitHub Pages.
- [ ] Enable private security reporting.
- [ ] Add repository topics and description.
