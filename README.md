# code-verification-lean4

Research and tooling to verify network/security protocol implementations in any codebase using Lean 4.

**Status:** early development (`0.x`) — the public API is not yet stable.

<!-- Two things most READMEs omit, and the two most worth keeping: an explicit purpose statement and
     a status line. Both are above. Do not delete them.
     GitHub auto-generates a table of contents from headings via the Outline menu — do not hand-maintain
     one until this file exceeds ~100 lines. Use relative links; absolute links break in clones. -->

## Why this exists

Protocol specifications (TLS, QUIC, SSH, Signal, MQTT, etc.) are routinely proven correct in one tool and implemented in another, and nobody checks the gap. This project tests the hypothesis that Lean 4 cannot verify protocols, and therefore cannot check code implementations of them. The aim is to find out whether that is true, and if not, to build tooling that verifies protocol implementations in any codebase.

**Current phase:** Phase 1, evidence gathering. Research lives in [`research/`](research/README.md). No analysis or conclusions yet.

## Quick start

No runnable tooling yet. Start with [`research/README.md`](research/README.md) and [`handoff.md`](handoff.md).

## Usage

Not applicable until tooling exists.

## Documentation

| Document | Answers |
|---|---|
| [PRD.md](PRD.md) | Why we are building this, and for whom |
| [TRD.md](TRD.md) | How it is built, and the requirement traceability matrix |
| [WIREFRAME.md](WIREFRAME.md) | Screen inventory and layouts |
| [DESIGN.md](DESIGN.md) | Design tokens and visual rules |
| [docs/index.md](docs/index.md) | Full documentation map and the architecture repository |
| [docs/adr/](docs/adr/) | Architecture decision records |

## Development

```bash
scripts/bootstrap     # install dependencies
scripts/setup         # bootstrap + local config
npm check     # lint, typecheck and test — the same gate CI runs
```

`package.json` is authoritative for available commands.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). This project is test-first: write the failing test before the
implementation. Commits follow Conventional Commits.

## Security

See [SECURITY.md](SECURITY.md). Do not open a public issue for a vulnerability.

## Support

See [SUPPORT.md](SUPPORT.md).

## License

Apache-2.0 — see [LICENSE](LICENSE). Copyright (c) 2026 Profit Rise Consulting.
