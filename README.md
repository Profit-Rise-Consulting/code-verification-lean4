# code-verification-lean4

Can Lean 4 verify network and security protocols, and check that real code implements them correctly, in any codebase?

**Status:** Phase 1b (evidence gathering and reference dossiers). No verdict yet. **Start here:** [handoff.md](handoff.md).

## Why this exists

Protocols such as TLS, QUIC, MQTT and Raft are often proven correct in one tool and implemented in another, and the gap between the two is where many vulnerabilities live. Profit Rise Consulting is testing the hypothesis that "LEAN4 cannot verify protocols AND therefore unable to check code implementation of these protocols". If the hypothesis fails, the project will build tooling that does what it says Lean cannot.

## Phases

| Phase | Goal | Status |
|---|---|---|
| 1 / 1b | Evidence corpus that is read in full and preserves conflicts, plus dossiers on prior protocol and algorithm verification work | in progress |
| 2 | Multi-model council debate on each sub-claim of the hypothesis | not started |
| 3 | Pilot implementation, if the verdict supports it | not started |

## Documentation

| Document | Answers |
|---|---|
| [handoff.md](handoff.md) | Where are we, and what does the next agent do? |
| [phase1-research.md](phase1-research.md) | What did Phase 1 gather, what is missing, and what is the Phase 2 brief? |
| [research/CONVENTIONS.md](research/CONVENTIONS.md) | What counts as evidence, and what are sub-claims H1-H5 and D? |
| [research/](research/README.md) | Topic evidence files |
| [research/references/](research/references/TEMPLATE.md) | Reference dossiers on prior work |
| [docs/phase2/council-kit.md](docs/phase2/council-kit.md) | How will the council debate run? |
| [docs/handoff/original-requests.md](docs/handoff/original-requests.md) | What exactly did the owner ask for? |
| [PRD.md](PRD.md), [TRD.md](TRD.md) | Requirements and how they are checked |
| [docs/adr/](docs/adr/) | Decisions |

## Development

```bash
scripts/test     # the same gate CI runs
```

Requires only Python 3 and bash.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). Evidence must be fully read or code-inspected; see [ADR-0003](docs/adr/0003-full-read-evidence-standard.md).

## Security

See [SECURITY.md](SECURITY.md). Do not open a public issue for a vulnerability.

## Support

See [SUPPORT.md](SUPPORT.md).

## License

Apache-2.0. See [LICENSE](LICENSE). Copyright (c) 2026 Profit Rise Consulting.
