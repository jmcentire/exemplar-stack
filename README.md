# Exemplar Stack

Exemplar is a component stack for contract-driven, observable, human-governed
automation in small-business operator workflows. Reeve currently contains
first implementations of some stack disciplines, but those disciplines are
stack components, not Reeve features.

The integration stance is proactive: safety, routing, anomaly, authority,
audit, and smoke controls are wired before a customer-visible failure proves
the need. Implementation can phase by maturity, but these controls are part of
the gold-standard operating environment.

For an infrastructure-team handoff that explains the operational invariants,
composition paths, failure semantics, and capability introduced by the tool
suite, read [docs/infrastructure-handoff.md](docs/infrastructure-handoff.md).
For the local `~/Code` inventory that separates core Exemplar tools from
supporting, adjacent, research, and non-tool repos, read
[docs/code-inventory.md](docs/code-inventory.md).

## Component Map

| Component | Language | Charter | Primary consumers | ADR |
| --- | --- | --- | --- | --- |
| Constrain | Python | Interview and synthesize stack artifacts from problem intent. | Pact, Ledger, Arbiter, Baton | `~/Code/constrain/README.md` |
| Pact | Python | Contract-first decomposition, tests, and agent implementation pipeline. | Reeve, Sentinel, Arbiter | `~/Code/pact/README.md` |
| Reeve | TypeScript | Operator-facing business automation and first integration host. | Operators, Baton, Tessera | `~/Code/reeve/docs/stack-roadmap.md` |
| Baton | Python | Circuit orchestration, adapter control, taint scanning, canary routing. | Reeve, stack-smoke | `~/Code/baton/CLAUDE.md` |
| Ledger | Python | Field classification and obligation registry. | Baton, Reeve, Sentinel | `~/Code/ledger/CLAUDE.md` |
| Arbiter | Python | Access auditing, consistency analysis, blast-radius classification, and trust enforcement. | Baton, Ledger, Sentinel | `~/Code/arbiter/README.md` |
| Sentinel | Python | PACT-key attribution and enforcement severity. | Baton, Reeve | `~/Code/sentinel/design.md` |
| Tessera | Rust | Self-validating executable document and hash-chain evidence format. | Reeve, scram, witness | `~/Code/tessera/README.md` |
| Chronicler | Python | Event collection and story assembly from spans, logs, webhooks, and incidents. | Reeve, Sentinel, Vigil | `~/Code/chronicler/README.md` |
| Stigmergy | Python | Organizational pattern discovery over work artifacts and correlated stories. | Chronicler, Apprentice, operators | `~/Code/stigmergy/README.md` |
| Apprentice | Python | Distill repeated frontier-model tasks into cheaper local models with quality gates. | Reeve, Chronicler, Stigmergy | `~/Code/apprentice/README.md` |
| Signet | Rust | Sovereign credential vault, proof, and agent authority policy. | Reeve, witness, operator identity | `~/Code/signet/README.md` |
| Cartographer | Python | Stack adoption, discovery, and compatibility checks for existing codebases. | Infrastructure team, CI | `~/Code/cartographer/README.md` |
| aegis | TypeScript + Python | Hot-path resource budgets and egress wrappers with golden vectors plus differential fuzzing. | Reeve, Baton | `~/Code/aegis/ADR-001-extraction.md` |
| covenant | TypeScript + Python | Zod-canonical contracts exported to committed JSON Schema for Python consumers. | Reeve, Baton, Ledger, Sentinel | `~/Code/covenant/ADR-001-extraction.md` |
| vigil | Python | Off-path anomaly detection using rolling quantile baselines over stack event streams. | Baton, Reeve dashboard | `~/Code/vigil/ADR-001-extraction.md` |
| scram | Python | Emergency kill switch and read-only/quarantine actions. | Reeve, Baton, witness | `~/Code/scram/ADR-001-extraction.md` |
| witness | TypeScript | Human-in-the-loop decisions and two-person approval. | Reeve, scram | `~/Code/witness/ADR-001-extraction.md` |

## Dependency Shape

```text
Constrain ──synthesizes──> Pact ──contracts/tests──> Reeve

Reeve ──emits events──> Baton ──taint/attribution──> Sentinel
  │                       │
  │                       ├──reads classifications──> Ledger
  │                       └──forwards spans/findings──> Arbiter
  │
  ├──writes audit evidence──> Tessera
  ├──emits stories/events───> Chronicler ──patterns──> Stigmergy
  ├──uses credentials/proofs──> Signet
  ├──feeds repeatable tasks───> Apprentice
  ├──uses hot-path budgets──> aegis
  ├──validates contracts───> covenant
  ├──surfaces anomalies────> vigil
  ├──registers conditions──> scram
  └──routes human review───> witness

Cartographer scans existing systems and drafts Constrain/Pact/Ledger/Arbiter/
Baton/Sentinel artifacts before a component is fully onboarded.
```

## Production-Stability Disciplines

- Budget every hot-path external resource through aegis.
- Validate wire contracts through covenant before cross-component egress;
  TypeScript Zod contracts are canonical and JSON Schema export drift is a CI failure.
- Use Ledger for data obligations and Arbiter for access/trust/blast-radius
  findings derived from observed behavior.
- Emit component health through peer APIs rather than a shared health table.
- Correlate raw events into stories through Chronicler before feeding pattern
  consumers such as Vigil, Stigmergy, and Apprentice.
- Use Cartographer during adoption and CI compatibility checks to discover
  missing stack artifacts before runtime.
- Keep stack-wide and continuous smoke assertions in `~/Code/stack-smoke`.
- Keep Reeve-specific migrations to extracted libraries in Wave 3, after
  component ADRs and skeletons land.

## Current Status

- `aegis`, `covenant`, `vigil`, `scram`, and `witness` are standalone public
  repos with `v0.1.0` releases.
- `baton`, `ledger`, `arbiter`, `chronicler`, `stigmergy`, `cartographer`,
  `stack-smoke`, and this overview repo are part of the broader Exemplar
  tooling surface.
- `baton-stack` is deployed on Fly and participates in live stack-smoke via
  `/api/snapshot`.
- Reeve is the first production integration host. It is deployed to staging and
  production with smoke checks for liveness, readiness, audit-chain shape,
  stack mode, and registry population.
- The next phase is composition: migrate Reeve to consume the extracted
  libraries, make Scram dispatchers real, surface Vigil in the operator
  dashboard, keep continuous smoke running, and promote `stack-smoke` into a
  full multi-service scenario.
