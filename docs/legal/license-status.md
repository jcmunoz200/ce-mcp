# License status

## Current status

**No license is granted.** This repository is a public design preview, not an
open-source project.

| Item | Status |
| --- | --- |
| Implementation | Does not exist yet |
| Documentation | Published for viewing |
| License | Not chosen |
| Code contributions | Paused |
| Open source | No |
| Source-available | Yes, for viewing |
| Warranty | None |

Copyright (c) 2026 ce-mcp contributors. All rights reserved.

## What you may do

- View the repository on GitHub or in a clone.
- Read the documentation.
- Open an issue to discuss the design.
- Quote short passages with attribution for review or commentary, as permitted
  by applicable law.

## What you may not do

Without prior written permission:

- copy, modify or create derivative works from the documentation;
- redistribute the documentation or a modified version;
- use the documentation to train a model or build a commercial product;
- sublicense or sell the contents;
- use the project name or trademarks in a way that implies endorsement.

These restrictions apply to the documentation and specifications in this
repository. They do not apply to Cheat Engine, which is governed by its own
terms.

## Why no license yet

The license for the implementation is a separate decision from the decision to
publish the design.

The project has not chosen it because:

- the implementation does not exist yet;
- the dependency set is not final;
- it is not clear whether any component will need code derived from a
  third-party project;
- a license chosen now might not fit the first code release.

Publishing the design without a license is deliberate. It lets people review the
work without granting rights the project is not ready to grant.

## What "source-available" means here

"Source-available" means the source files can be viewed. It does **not** mean
they meet the Open Source Initiative's definition of open source. The OSI
definition requires, among other things, free redistribution and derived works.
Those rights are not granted.

GitHub may show a license badge or a "View license" link. `LICENSE.md` is the
authoritative statement: no license is granted.

## Path to open source

Before the first code pull request is merged:

1. Choose the implementation license.
2. Add the full license text.
3. Update `README.md`, `LICENSE.md`, `CONTRIBUTING.md` and package metadata.
4. Confirm dependency license compatibility.
5. Record the decision in an ADR that supersedes
   [ADR-0007](../decisions/ADR-0007-docs-license.md).
6. Enable code scanning and dependency review.

The documentation may be relicensed separately. A documentation license such as
CC BY 4.0 and a code license such as MIT or Apache-2.0 are plausible, but no
choice has been made.

## Contributions

- Issues and design feedback are welcome.
- Code pull requests are paused.
- By submitting a design proposal, you agree that it may be discussed
  publicly. Do not submit text you do not have the right to share. Because no
  license is granted yet, the maintainer will ask before incorporating
  substantial third-party text.
- Do not submit code or documentation copied from Cheat Engine or another
  project.

See [CONTRIBUTING.md](../../CONTRIBUTING.md).

## Third-party material

This repository does not include Cheat Engine binaries, source code, language
files, images or documentation. It links to the official project and records
factual API names for interoperability.

Other third-party projects are referenced, not vendored. See
[Third-party notices](third-party.md).

## FAQ

### Can I use this in my project?

Not without permission. There is no license.

### Can I fork it?

GitHub's terms allow forking within GitHub, but that does not grant a license to
use, modify or distribute the contents. Do not rely on a fork as a legal
license.

### Will it become open source?

The intent is to choose a license before code is published. The specific license
is undecided.

### Does this affect Cheat Engine?

No. Cheat Engine has its own terms. This repository does not grant or restrict
rights to Cheat Engine.

### Is this legal advice?

No. Consult a qualified lawyer for your situation.

## Related

- [LICENSE.md](../../LICENSE.md)
- [ADR-0007](../decisions/ADR-0007-docs-license.md)
- [Open-source readiness](../migration/07-open-source.md)
