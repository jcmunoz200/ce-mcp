# Governance

## Current model

`ce-mcp` is currently maintained by a single maintainer (see
[MAINTAINERS.md](MAINTAINERS.md)). The maintainer makes final decisions on
scope, architecture, licensing and releases.

The project is in a design-preview phase. Governance will become more formal
when code contributions open.

## Decision process

Architecture decisions are recorded as ADRs in `docs/decisions/`:

1. Anyone may propose a decision by opening a Design proposal issue.
2. The maintainer discusses the proposal and requests an ADR if the change is
   architectural.
3. The ADR is merged with status `accepted`, `rejected` or `superseded`.
4. Accepted ADRs are linked from `docs/migration/04-decisions.md`.

## License decisions

The implementation license is not chosen yet. Before code contributions are
accepted:

- the maintainer selects a license;
- `LICENSE.md`, `README.md`, `CONTRIBUTING.md` and the package metadata are
  updated;
- the decision is recorded in an ADR.

Changing the license after external code contributions requires permission from
all contributors or a contributor license agreement. The project intends to
avoid that situation by choosing the license before code is merged.

## Releases

- Documentation releases follow the roadmap phases.
- Implementation releases will use Semantic Versioning.
- Release notes are generated from `CHANGELOG.md`.
- Release artifacts will include checksums.

## Maintainers

Maintainers are listed in [MAINTAINERS.md](MAINTAINERS.md). A new maintainer is
added by consensus of the existing maintainers, based on sustained,
high-quality contributions and adherence to the code of conduct.

## Changes to this document

Changes to governance require an ADR and maintainer approval.
