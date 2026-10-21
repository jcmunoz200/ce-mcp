# ADR-0007: Publish the documentation without a license

- **Status:** Accepted
- **Date:** 2026-10-21
- **Supersedes:** none
- **Superseded by:** a future implementation-license ADR

## Context

The project wants a public repository now so the design and migration can be
reviewed. The implementation license has not been chosen.

Publishing a repository publicly does not grant rights. Without a license,
default copyright applies. Calling such a repository "open source" would be
incorrect.

The project could:

1. choose a permissive license immediately;
2. publish privately until the license is chosen;
3. publish publicly with an explicit "no license" notice.

## Decision

Publish the documentation as a **public design preview** with no license
granted.

- `LICENSE.md` states "All rights reserved" and "No license is granted at this
  time."
- `README.md` and `docs/legal/license-status.md` repeat the status.
- The repository is not described as open source.
- Code pull requests are paused.
- The implementation license must be chosen before F0 code is merged.
- A future ADR will supersede this one when the license is selected.

The documentation itself is authored for this project. It does not copy Cheat
Engine documentation or third-party prose.

## Consequences

### Positive

- The design can be reviewed now.
- The license decision is not rushed.
- No accidental grant of rights over future code.
- The project can choose a license that fits its dependencies.

### Negative

- The repository is source-available, not open source.
- Others cannot legally reuse the documentation without permission.
- GitHub may show an unclear license status.
- Code contributions are blocked, which slows community involvement.
- Some users will ignore the notice.

### Neutral

- The maintainer can relicense their own documentation later.
- A future license change cannot retroactively remove rights already granted,
  but no rights are granted now.

## Alternatives considered

1. **MIT now.** Rejected: premature; the implementation may include components
   with different licensing constraints.
2. **CC BY 4.0 for documentation and MIT for code.** Reasonable, but still a
   license decision that should be made with the implementation plan.
3. **Private repository.** Rejected: loses early review and the public record of
   the migration.
4. **No LICENSE file at all.** Rejected: ambiguous; an explicit notice is
   clearer.

## References

- [License status](../legal/license-status.md)
- [Open-source readiness](../migration/07-open-source.md)
- [Contributing](../../CONTRIBUTING.md)
