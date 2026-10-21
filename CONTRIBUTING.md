# Contributing

Thank you for your interest in `ce-mcp`.

## Current phase: design preview

The repository currently contains design and migration documentation. There is
no implementation and **no license for code contributions yet**.

- **Issues and design feedback are welcome.**
- **Code pull requests are paused** until the implementation license is chosen.
- Documentation corrections are welcome as issues. Do not submit large copied
  passages from Cheat Engine or third-party documentation.

## How to contribute a design change

1. Open an issue using the **Design proposal** template.
2. Describe the problem, the proposed change and the alternatives considered.
3. If the change affects architecture, transports, tool contracts, security or
   licensing, it needs an ADR in `docs/decisions/`.
4. Wait for maintainer feedback before writing a large document.

Architecture decisions use a lightweight ADR process:

- one file per decision: `ADR-NNNN-short-title.md`;
- status: proposed, accepted, superseded or rejected;
- context, decision, consequences and alternatives;
- link the ADR from `docs/migration/04-decisions.md` and `docs/SUMMARY.md`.

## Documentation style

- English only.
- Prefer short sentences and concrete examples.
- Distinguish **observation**, **inference** and **unknown**.
- Link to official sources instead of copying them.
- Do not include Cheat Engine binaries, screenshots, language files, tutorials
  or long excerpts from its documentation.
- Do not include game-specific cheats, anti-cheat bypass instructions or
  material that violates a third party's terms.
- Use relative links inside the repository.
- Run `markdownlint` before opening an issue with a documentation patch.

## Local checks

```bash
npx --yes markdownlint-cli2 "**/*.md"
mdbook build docs
```

The CI uses mdBook 0.5.4 and the configuration in `docs/book.toml`.

## Commit style

Use [Conventional Commits](https://www.conventionalcommits.org/):

```text
docs: clarify scan profiles
docs(adr): add transport decision
fix(docs): repair broken link
```

## Pull requests

Code pull requests will be rejected until the implementation license is
selected. When the license is in place, the pull request template will require:

- a linked issue or ADR;
- tests for behavior changes;
- documentation updates;
- `markdownlint` and link checks passing;
- no Cheat Engine assets or copied documentation.

## Security

Do not report vulnerabilities in public issues. See [SECURITY.md](SECURITY.md).

## Code of conduct

Participation is governed by [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md).
