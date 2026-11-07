# Contributing to code-verification-lean4

Thanks for helping. This document is the human workflow; `AGENTS.md` is the machine-readable
equivalent for coding agents, and the two must not contradict each other.

## Ways to contribute

Bug reports, reproductions, documentation fixes and reviews are all as valuable as code. Open an
issue using one of the [issue forms](.github/ISSUE_TEMPLATE) before starting substantial work.

**Do not report a security vulnerability in a public issue.** See [SECURITY.md](SECURITY.md).

## Find an issue

Issues labelled `good first issue` and `help wanted` are ready to pick up. Comment on the issue to
claim it so two people do not duplicate work.

## Development environment

```bash
git clone https://github.com/Profit-Rise-Consulting/code-verification-lean4.git
cd code-verification-lean4
scripts/test
```

Requires only Python 3 and bash. There is no Lean toolchain yet ([ADR-0002](docs/adr/0002-research-first-scope.md)).

## Evidence-first workflow

This is a research repository; its "tests" are the corpus gates.

1. Read [research/CONVENTIONS.md](research/CONVENTIONS.md) before adding evidence.
2. Add evidence only from fully read sources or inspected code at a recorded commit.
3. For a new reference, copy [research/references/TEMPLATE.md](research/references/TEMPLATE.md), add its `_index` JSON, and run `scripts/build-catalogue`.
4. Run `scripts/test` until it exits 0.

## Commits

[Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/):

```
<type>[optional scope][optional !]: <description>
```

`feat` for features, `fix` for bug fixes; also `build`, `chore`, `ci`, `docs`, `perf`, `refactor`,
`revert`, `style`, `test`. Breaking changes use `!` before the colon or a `BREAKING CHANGE:` footer
in uppercase.

Use the requirement id as the scope where one applies: `feat(FR-001): add email validation`.

## Pull request lifecycle

- Open a draft pull request early if you want feedback on direction; mark it ready when it is.
- Keep the diff narrow — one concern per pull request. Large refactors go in their own PR.
- Fill in the [pull request template](.github/pull_request_template.md), including `Closes #NNN`.
- Run the same gate CI runs before pushing: `scripts/test`.
- Expect a first review within a few working days. Ping the thread once after that; do not ping
  repeatedly.
- A stalled pull request may be taken over or closed for inactivity after a comment asking about it.

## Pull request checklist

- [ ] `scripts/test` passes locally
- [ ] New evidence is [FULL] or [CODE]; excerpts are under Unverified leads
- [ ] Documentation updated: PRD, TRD matrix as applicable
- [ ] An ADR added if an architecturally significant decision was made
- [ ] No secret values committed

## Documentation changes

The document spine is validated in CI. If you change a requirement, update the traceability matrix in
[TRD.md](TRD.md) section 4 in the same pull request. Unresolved clarification markers (see
[docs/index.md](docs/index.md)) fail the build.

## Code of conduct

Participation is governed by [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md).
