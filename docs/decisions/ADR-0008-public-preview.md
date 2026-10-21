# ADR-0008: Publish a documentation-only public preview

- **Status:** Accepted
- **Date:** 2026-10-21
- **Supersedes:** none
- **Superseded by:** none

## Context

The design and migration work is substantial and useful on its own. The
implementation is not started, and the license is not chosen.

Publishing a repository with placeholder code, an unbuildable `Cargo.toml` or a
half-working bridge would:

- attract bug reports for code that is not ready;
- create expectations that cannot be met;
- complicate the license decision;
- mix design review with implementation review.

Publishing nothing until the code is ready would lose the opportunity to get
feedback on the architecture and tool contract.

## Decision

Publish a **documentation-only** repository now.

- No `Cargo.toml`.
- No Rust source.
- No Lua bridge code.
- No binaries, installers or release artifacts.
- The tool catalog lives in `spec/` as data, not executable code.
- The documentation site is built from `docs/` with mdBook.
- The repository is labelled "design preview" in the README, the book and the
  changelog.

Code starts in F0, after the implementation license is selected.

## Consequences

### Positive

- Early review of the architecture and tool contract.
- A clear, citable record of the migration.
- No unbuildable code or false release signals.
- The license decision stays separate from the design.
- Contributors can discuss the design without a code review bottleneck.

### Negative

- The repository cannot yet be used.
- Some visitors expect code and leave.
- Documentation can drift from the eventual implementation.
- There is no test suite proving the design works.

### Neutral

- The design is explicitly provisional.
- The journal records uncertainty and open questions.
- A future release will add code and update the README.

## Alternatives considered

1. **Publish a code skeleton.** Rejected: adds noise and license ambiguity
   without validating anything.
2. **Wait for F0.** Rejected: loses early feedback and the public migration
   record.
3. **Publish the bridge only.** Rejected: still requires a license and creates
   an unsupported partial product.
4. **Publish as a blog post.** Rejected: a repository is easier to update,
   review and link.

## References

- [Roadmap](../../ROADMAP.md)
- [Open-source readiness](../migration/07-open-source.md)
- [ADR-0007](ADR-0007-docs-license.md)
