# Exemplar Stack

Exemplar is a component stack for contract-driven, observable, human-governed
automation in small-business operator workflows. Reeve currently contains
first implementations of some stack disciplines, but those disciplines are
stack components, not Reeve features.

## Component Map

| Component | Language | Charter | Primary consumers | ADR |
| --- | --- | --- | --- | --- |
| Reeve | TypeScript | Operator-facing business automation and first integration host. | Operators, Baton, Tessera | `~/Code/reeve/docs/stack-roadmap.md` |
| Baton | Python | Circuit orchestration, adapter control, taint scanning, canary routing. | Reeve, stack-smoke | `~/Code/baton/CLAUDE.md` |
| Ledger | Python | Field classification and obligation registry. | Baton, Reeve, Sentinel | `~/Code/ledger/CLAUDE.md` |
| Sentinel | Python | PACT-key attribution and enforcement severity. | Baton, Reeve | TBD |
| Tessera | TBD | Append-only audit evidence and hash-chain integrity. | Reeve, scram, witness | TBD |
| aegis | TypeScript + Python | Hot-path resource budgets and egress wrappers with golden vectors plus differential fuzzing. | Reeve, Baton | `~/Code/aegis/ADR-001-extraction.md` |
| covenant | TypeScript + Python | Zod-canonical contracts exported to committed JSON Schema for Python consumers. | Reeve, Baton, Ledger, Sentinel | `~/Code/covenant/ADR-001-extraction.md` |
| vigil | Python | Off-path anomaly detection using rolling quantile baselines over stack event streams. | Baton, Reeve dashboard | `~/Code/vigil/ADR-001-extraction.md` |
| scram | Python | Emergency kill switch and read-only/quarantine actions. | Reeve, Baton, witness | `~/Code/scram/ADR-001-extraction.md` |
| witness | TypeScript | Human-in-the-loop decisions and two-person approval. | Reeve, scram | `~/Code/witness/ADR-001-extraction.md` |

## Dependency Shape

```text
Reeve ──emits events──> Baton ──taint/attribution──> Sentinel
  │                       │
  │                       └──reads classifications──> Ledger
  │
  ├──writes audit evidence──> Tessera
  ├──uses hot-path budgets──> aegis
  ├──validates contracts───> covenant
  ├──surfaces anomalies────> vigil
  ├──registers conditions──> scram
  └──routes human review───> witness
```

## Production-Stability Disciplines

- Budget every hot-path external resource through aegis.
- Validate wire contracts through covenant before cross-component egress;
  TypeScript Zod contracts are canonical and JSON Schema export drift is a CI failure.
- Emit component health through peer APIs rather than a shared health table.
- Keep stack-wide smoke assertions in `~/Code/stack-smoke`.
- Keep Reeve-specific migrations to extracted libraries in Wave 3, after
  component ADRs and skeletons land.

## Current Work Split

- Track 1: aegis, covenant, and vigil ADRs are locked on disk.
- Track 2: scram and witness after Track 1 contracts stabilize.
- Track 3: Reeve ledger publish, Baton Reeve smoke/canary config,
  stack-smoke scaffold, and this stack overview.

The track/wave language is temporary project coordination: Track 1 extracts
shared architecture components, Track 2 builds emergency/human coordination
components, and Track 3 wires operational integration. Wave 3 is where Reeve
migrates to the extracted libraries after their ADRs and skeletons land.

## Track 1 Status

- `~/Code/aegis/ADR-001-extraction.md` is locked. The implementation derives
  from Reeve's working private module at `~/Code/reeve/src/observability/aegis/`.
  The standalone repo is not git-initialized yet.
- `~/Code/covenant/ADR-001-extraction.md` is locked. The implementation derives
  from Reeve's working private Zod-native module at `~/Code/reeve/src/covenant/`.
  The standalone repo is not git-initialized yet.
- `~/Code/vigil/ADR-001-extraction.md` is locked. The implementation is net-new;
  Reeve's `src/tracing/` is an input source, not a private vigil implementation.
  The standalone repo is not git-initialized yet.
