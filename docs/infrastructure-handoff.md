# Exemplar Stack Infrastructure Handoff

This document explains the Exemplar stack as an operational control system:
what invariants it enforces, which tools enforce them, how those tools compose,
and what capability the stack introduces for AI-assisted operations.

It is written for infrastructure engineers who need to operate, extend, or
review the stack. It is not a product overview.

For the companion inventory of local `~/Code` repos and how each one relates to
Exemplar/Reeve, see [code-inventory.md](code-inventory.md).

## Executive Summary

Exemplar is a stack for running AI-assisted business automation without relying
on trust in the agent, the developer, or the happy path. The stack makes unsafe
operations visible, bounded, contract-checked, auditable, interruptible, and
human-governed.

The central design choice is to treat AI-facing infrastructure as a system of
enforced invariants:

- Every blocking external call has a budget.
- Every cross-boundary payload has a contract.
- Every data field has declared obligations.
- Every egress path can be routed, checked, paused, or rolled back.
- Every important action is observable.
- Every human decision is recorded with rationale.
- Every emergency action has a defined authority path.
- Every deploy and incident path has a smoke signal that can fail closed.

The tools are intentionally small and composable. Reeve is the first full
integration host, but the components are stack-level infrastructure rather than
Reeve-specific features.

## Proactive Safety Stance

The stack treats safety controls as design inputs, not incident aftermath.
Absence of a known user harm is not evidence that a guardrail is speculative.
Vigil, Baton, continuous smoke, Scram, Witness, Tessera, Ledger, Arbiter,
Chronicler, Stigmergy, Apprentice, Signet, Cartographer, Constrain, Pact,
Aegis, and Covenant are part of the gold-standard operating environment
because each closes a distinct failure mode before it becomes customer-visible.

Infrastructure teams should therefore evaluate integrations by whether they
make an invariant enforceable, observable, auditable, or reversible. A control
can be phased in by implementation maturity, but it should not be deferred
until the stack has already produced the harm it was meant to prevent.

## Current Release Surface

The following repos are the current operational surface from the May 2026
release pass:

| Repo | Version / state | Role |
| --- | --- | --- |
| `aegis` | `v0.1.0` | Hot-path resource budgets for TypeScript and Python. |
| `covenant` | `v0.1.0` | Contract validation and policy for TypeScript and Python consumers. |
| `vigil` | `v0.1.0` | Off-path anomaly detection and forensic query. |
| `scram` | `v0.1.0` | Emergency kill-switch registry, evaluator, API, CLI, and read-only override. |
| `witness` | `v0.1.0` | Human-in-the-loop decisions and two-person approval. |
| `baton` | `v0.3.5`, deployed `baton-stack` | Circuit orchestration, adapter control, taint scanning, canary routing. |
| `ledger` | `v0.2.2` | Schema registry and data obligation manager. |
| `arbiter` | `v0.2.1` | Access auditing, consistency analysis, blast-radius classification, and trust enforcement. |
| `chronicler` | `v0.3.0` | Event collection and story assembly from operational signals. |
| `stigmergy` | `v0.1.2` | Pattern discovery over work artifacts, stories, and coordination signals. |
| `cartographer` | `v0.1.2` | Stack adoption, discovery, and compatibility reporting. |
| `constrain` | `v0.5.0` | Structured problem interview and artifact synthesis. |
| `pact` | `v0.14.1` | Contract-first decomposition, executable tests, and agent implementation. |
| `apprentice` | `v0.3.2` | Model distillation, routing, and quality-gated local inference. |
| `signet` | public Rust stack | Sovereign vault, credentials, proofs, and agent authority policy. |
| `stack-smoke` | `v0.1.3` | Continuous smoke harness with local prerequisite checks, live Reeve/Baton smoke checks, Baton version drift check, and cross-component target flow. |
| `exemplar-stack` | `v0.1.0+` | Architecture map and coordination docs. |
| `reeve` | private, deployed | First production integration host. |

Reeve is currently deployed to staging and production with live smoke checks for
liveness, readiness, audit-chain shape, stack mode, and registry population.
Baton is deployed as the `baton-stack` Fly app and exposes `/api/snapshot` plus
`/v1/about` for continuous smoke and version drift checks.

## Component Responsibilities

The stack has two classes of components: operational primitives and integration
hosts.

| Component | Responsibility | Operational boundary |
| --- | --- | --- |
| Constrain | Intent capture and stack artifact synthesis. | Owns interviews and exports initial Pact, Ledger, Arbiter, Baton, and Sentinel scaffolds. |
| Pact | Contract-first build pipeline. | Owns decomposition, interface contracts, executable tests, and agent implementation gates. |
| Reeve | Operator-facing automation and first full integration host. | Owns business workflows, route handlers, operator UI, and first production proof. |
| Baton | Circuit and adapter control plane. | Owns traffic routing, health, canary routing, taint scanning, and egress config consumption. |
| Ledger | Schema and obligation registry. | Owns field classifications, retention/masking/export obligations, and downstream config generation. |
| Arbiter | Trust and blast-radius analysis. | Owns access auditing, observed-vs-declared consistency checks, trust scoring, and blast-radius findings. |
| Sentinel | Attribution and severity enforcement. | Owns PACT-key attribution and enforcement severity for data movement. |
| Tessera | Audit evidence and hash-chain integrity. | Owns append-only evidence and integrity checks for operational actions. |
| Chronicler | Event collection and story assembly. | Owns correlation of logs, spans, webhooks, and incidents into bounded operational stories. |
| Stigmergy | Organizational pattern discovery. | Owns cross-source signal processing for coordination gaps, dependency risks, and structural patterns. |
| Apprentice | Model distillation and quality-gated routing. | Owns when repeated model tasks move from frontier APIs to local models and when they fall back. |
| Signet | Credential, proof, and authority substrate. | Owns scoped credentials, selective disclosure, vault-backed authorization, and agent authority policy. |
| Cartographer | Adoption and compatibility discovery. | Owns scans that draft stack artifacts and report missing or incompatible integration surfaces. |
| Aegis | Resource-budget primitive. | Owns timeout/fallback semantics around blocking calls. |
| Covenant | Contract validation primitive. | Owns runtime shape validation and violation policy. |
| Vigil | Pattern-divergence detector. | Owns anomaly detection and forensic query over event streams. |
| Scram | Emergency kill-switch primitive. | Owns kill-condition evaluation and emergency action authority. |
| Witness | Human decision primitive. | Owns approval, rationale, and two-person coordination. |
| Stack Smoke | Cross-component smoke harness. | Owns assertions that prove multiple components work together. |

The important boundary is that Reeve uses these controls; it should not be the
permanent owner of them. Reeve was the proving ground. The extracted repos are
the reusable stack surface.

## Lifecycle View

The tools cover the full lifecycle, not only runtime operations:

| Phase | Tools | Capability |
| --- | --- | --- |
| Discover existing systems | Cartographer | Scan code/backends and draft missing stack artifacts. |
| Capture intent | Constrain | Turn problem interviews into component maps, trust policies, schema hints, and downstream scaffolds. |
| Build against contracts | Pact, Covenant | Decompose, generate contracts/tests, validate runtime payloads, and prevent interface drift. |
| Declare data obligations | Ledger | Classify fields and export masking, taint, retention, and severity rules. |
| Route and control services | Baton, Aegis | Control egress, canaries, health, and per-call budgets. |
| Establish authority | Signet, Witness | Scope credentials/proofs and route human approval. |
| Observe and learn | Chronicler, Vigil, Stigmergy, Arbiter, Sentinel | Assemble stories, detect anomalies/patterns, compute trust/blast radius, and attribute production failures. |
| Emergency response | Scram, Tessera | Trigger kill paths and preserve tamper-evident evidence. |
| Cost/quality optimization | Apprentice | Distill repeatable frontier-model tasks into local models under measured quality gates. |
| Verify composition | Stack Smoke, Reeve smoke endpoints | Prove the stack works across component boundaries. |

## The Operational Invariants

### 1. Intent Must Become Machine-Readable Before Implementation

AI-assisted infrastructure fails when intent lives only in chat transcripts or
tribal memory. The invariant is: architecture starts as structured artifacts.

**Enforced by:** `cartographer`, `constrain`, `pact`

**Meaning in practice:**

- Cartographer discovers what an existing system already contains.
- Constrain interviews engineers and exports component maps, trust policies,
  Ledger hints, Pact tasks, and Baton scaffolds.
- Pact decomposes work into contracts and tests before agents implement.

**Why it matters for AI systems:**

Agents need stable artifacts more than they need long explanations. These tools
turn intent into files that can be reviewed, tested, regenerated, and consumed
by downstream automation.

### 2. Blocking Work Must Be Budgeted

Any call to a vendor, model, database, subprocess, or peer service can hang,
stall, or consume more resources than the operator intended. The invariant is:
blocking work must declare its budget before it starts.

**Enforced by:** `aegis`

**Meaning in practice:**

- A call declares `timeoutMs`, resource class, tags, and optional fallback.
- The wrapper enforces timeout behavior and records structured completion.
- TypeScript and Python implementations share golden vectors and differential
  fuzzing so behavior stays equivalent across language boundaries.

**Why it matters for AI systems:**

AI agents tend to compose calls dynamically. Without budgets, an agent can turn
a slow external dependency into a saturated worker pool. With `aegis`, the agent
or caller must make the operational cost explicit at the boundary.

### 3. Cross-Boundary Data Must Match a Contract

Payloads crossing trust boundaries need more than TypeScript types. They need
runtime validation, policy, and drift detection for non-TypeScript consumers.

**Enforced by:** `covenant`

**Meaning in practice:**

- Zod is the canonical TypeScript authoring format.
- JSON Schema artifacts are committed for Python and other consumers.
- Policy determines what happens when a contract is violated.
- Export drift is a CI concern rather than an operator surprise.

**Why it matters for AI systems:**

Agents need machine-readable structure. A contract lets an agent generate,
validate, and repair a payload without a human translating prose into shape.
The runtime still validates because agent confidence is not a safety boundary.

### 4. Data Obligations Must Be Declared Once and Propagated

Teams should not hand-maintain separate rules for masking, taint detection,
retention, classification, and egress. The invariant is: the schema registry is
the source of truth for data obligations.

**Enforced by:** `ledger`

**Meaning in practice:**

- Schemas declare field classifications and obligations.
- Contradictory obligations are caught at schema time.
- Exports feed downstream tools such as Baton egress configs and contract
  assertions.
- Reeve now publishes its ledger schemas into Ledger and exports Baton egress
  config through the release workflow.

**Why it matters for AI systems:**

Agents need to know what data they can move, summarize, persist, or reveal. A
machine-readable obligation registry prevents every agent prompt from becoming
the policy source of truth.

### 5. Access, Trust, and Blast Radius Must Be Computed From Evidence

Declared policy is not enough; the stack needs to compare declarations against
observed behavior. The invariant is: access and trust decisions should be based
on evidence, not only on intent.

**Enforced by:** `arbiter`

**Meaning in practice:**

- Arbiter consumes spans and access graphs outside the hot path.
- It checks whether nodes touched data they were allowed to touch.
- It compares what components claim against what adapters observed.
- It computes trust scores, taint state, and blast-radius findings.

**Why it matters for AI systems:**

Agents can generate plausible declarations that are still wrong. Arbiter gives
operators a separate evidence loop: what happened, what was allowed, and what
changed the blast radius.

### 6. Egress and Service Topology Must Be Controllable

When a service or integration misbehaves, the stack needs routing authority:
pause, canary, mask, fail over, or collapse to a mock.

**Enforced by:** `baton`

**Meaning in practice:**

- Topology is described as a circuit, not inferred from scattered code.
- Adapters own routing, health, canaries, and taint scanning.
- Reeve-specific Baton configs now exist for smoke endpoints and Ledger-derived
  egress.

**Why it matters for AI systems:**

AI workflows often depend on multiple external services. Baton gives operators
a control plane for those edges instead of asking an agent to improvise outage
handling inside business logic.

### 7. Routine Health Must Be Observable and Fail Closed

The stack needs a current view of whether critical components are healthy and
whether the system should run normally, pause, or become read-only.

**Enforced by:** Reeve stack-health, Baton health views, `stack-smoke`

**Meaning in practice:**

- Reeve records component observations and derives a stack mode.
- Smoke endpoints expose operational checks as HTTP assertions.
- `make pre-deploy` runs local gates before deploy.
- Production and staging smoke checks verify the live deployed surface.

**Why it matters for AI systems:**

Agents should not make write decisions during degraded modes. Stack mode gives
callers a simple operational signal: safe, paused, manual-only, or read-only.

### 8. Events Must Become Stories Before They Become Lessons

Raw logs and spans are necessary but not sufficient. The invariant is:
operational learning should happen over correlated stories, not isolated log
lines.

**Enforced by:** `chronicler`

**Meaning in practice:**

- Chronicler ingests webhooks, OTLP spans, log files, and Sentinel incidents.
- It assembles request, service, and journey stories with bounded memory and
  configurable timeouts.
- It emits completed stories to pattern consumers such as Stigmergy, Apprentice,
  and anomaly systems.

**Why it matters for AI systems:**

Agents reason better over causal narratives than raw event streams. Chronicler
turns operational exhaust into bounded context that can be audited, summarized,
and learned from.

### 9. Organizational Patterns Must Be Detectable Across Tools

Incidents often come from coordination failures rather than code defects. The
invariant is: the stack should notice structural patterns across work systems.

**Enforced by:** `stigmergy`

**Meaning in practice:**

- Stigmergy ingests GitHub, Linear, Slack, and correlated story signals.
- It uses an adaptive mesh to form, reinforce, and decay pattern categories.
- It surfaces coordination gaps, knowledge silos, dependency risks, and other
  cross-source findings.

**Why it matters for AI systems:**

AI assistants need more than production telemetry; they need organizational
context. Stigmergy gives them pattern signals about how work actually moves
through the team.

### 10. Existing Systems Must Be Discoverable Before They Are Governed

Stack adoption should not start with hand-authored perfect YAML. The invariant
is: existing services can be scanned, drafted, and compatibility-checked before
the team commits to enforcement.

**Enforced by:** `cartographer`

**Meaning in practice:**

- Cartographer scans source code and optional live backends.
- It drafts artifacts for Constrain, Pact, Ledger, Arbiter, Baton, and Sentinel.
- It reports missing or incompatible stack requirements in CI-friendly form.

**Why it matters for AI systems:**

Agents onboarding a service need a map of what exists before they can safely
change it. Cartographer gives them a discovery surface instead of asking them
to infer architecture from scattered files.

### 11. Model Cost Savings Must Be Quality-Gated

Replacing a frontier model with a cheaper local model is only safe when the
local model earns trust through measured correlation. The invariant is: model
routing changes are evidence-based.

**Enforced by:** `apprentice`

**Meaning in practice:**

- Apprentice collects frontier-model outputs as training/evaluation data.
- It compares local and remote results during reinforcement phases.
- Traffic shifts only when rolling quality thresholds hold.
- Regression sends traffic back to a safer phase.

**Why it matters for AI systems:**

The stack can reduce cost without turning quality into a guess. Agents and
operators get a measured route between frontier capability and local efficiency.

### 12. Credentials and Authority Must Be Scoped and Auditable

Agents should not hold broad secrets or act as unbounded representatives of the
user. The invariant is: credentials, proofs, and authority are scoped,
revocable, and auditable.

**Enforced by:** `signet`

**Meaning in practice:**

- Signet stores root credentials in a vault-backed trust hierarchy.
- Agents request scoped capabilities and selective disclosures.
- Proofs and credential use are auditable and revocable.
- Reeve uses Signet-shaped authority boundaries for sensitive integrations.

**Why it matters for AI systems:**

An AI agent needs enough authority to act, but not enough to become the root of
trust. Signet makes authority explicit instead of embedding secrets in prompts
or process environments.

### 13. Pattern Drift Must Be Detectable Below Alarm Thresholds

Not every incident starts as a single obvious failure. Many show up as unusual
clusters: timeouts, repeated contract violations, or tenant-specific outliers.

**Enforced by:** `vigil`

**Meaning in practice:**

- Vigil ingests event streams and trace samples.
- It maintains rolling baselines by `(component, op, tenant)`.
- It writes anomalies when observed behavior diverges from baseline.
- Operators query anomalies and dismiss or escalate with context.

**Why it matters for AI systems:**

Model and tool behavior can drift gradually. Vigil catches "this pattern is
weird now" before it becomes a declared outage or customer-visible failure.

### 14. Catastrophic Actions Need an Emergency Authority Path

Some failures require immediate shutdown, quarantine, rollback, or read-only
mode. The invariant is: emergency action cannot depend on ad-hoc operator
memory.

**Enforced by:** `scram`

**Meaning in practice:**

- Kill conditions are registered and evaluated on a loop.
- `SCRAM_FORCE_READONLY=1` is a boot-only failsafe.
- Actions include process exit, rollback, circuit break, tenant quarantine,
  and global read-only.
- V1 dispatchers are explicit stubs; they record intent and are ready for V2
  integration with Baton and a shared control plane.

**Why it matters for AI systems:**

If an AI workflow starts causing cross-tenant leakage, runaway writes, or
classification fail-closed cascades, the system needs a kill path that is faster
than a design meeting.

### 15. Human Judgment Must Be a First-Class Primitive

Some decisions should not be automated. The invariant is: human approval,
rationale, and second-operator requirements are infrastructure, not UI glue.

**Enforced by:** `witness`

**Meaning in practice:**

- Callers ask for decisions synchronously or asynchronously.
- Decisions capture kind, input, operator, rationale, output, and second
  operator when needed.
- Two-person approval is state-based through `context_hash`, not a naive time
  window.
- Surface delivery is ACK-based, so fallback routing is explicit.

**Why it matters for AI systems:**

Witness gives agents a safe way to stop and ask, and gives operators a durable
record of why a human allowed or rejected an action.

### 16. Cross-Component Behavior Must Be Tested as a Flow

Unit tests prove components. Smoke tests prove operational composition.

**Enforced by:** `stack-smoke`

**Meaning in practice:**

- The repo owns cross-boundary smoke assertions.
- Current V1 checks prerequisites and records the target Reeve -> Baton ->
  Sentinel -> Tessera path.
- The intended path drives a synthetic inbound message and verifies event
  emission, taint scanning, attribution, audit append, and health observations.

**Why it matters for AI systems:**

Agents need confidence that the environment they are acting inside is coherent.
Cross-stack smoke tests make that coherence testable rather than assumed.

## How the Tools Work Together

### Request Path

```text
Cartographer / Constrain / Pact
  |
  |-- discover existing system
  |-- synthesize stack artifacts
  |-- generate contracts and tests
  v
Operator or inbound event
  |
  v
Reeve
  |-- wraps external calls through aegis
  |-- validates egress and ingress through covenant
  |-- consults Ledger-derived obligations for data handling
  |-- emits operational traces, stories, and audit evidence
  |-- uses Signet-scoped credentials and authority proofs
  |-- routes repeatable model tasks through Apprentice where appropriate
  |
  v
Baton adapters
  |-- route or pause traffic
  |-- apply egress masking from Ledger
  |-- scan for taint fingerprints
  |-- forward observed behavior to Arbiter
  |
  v
Sentinel / Tessera / peer systems
```

**Failure semantics:**

- Aegis timeout: caller receives timeout/fallback behavior instead of hanging.
- Covenant violation: policy decides reject, warn, record, or fail closed.
- Baton unhealthy route: traffic can pause, roll back, or collapse to a safe
  alternate path.
- Ledger obligation conflict: schema validation fails before runtime.
- Arbiter finding: trust/blast-radius state changes and can gate later rollout
  or require human review depending on policy.

### Deploy Path

```text
Developer change
  |
  v
make pre-deploy
  |-- preflight environment checks
  |-- Ledger publish dry-run
  |-- lint, type-check, unit tests, build
  |
  v
Staging deploy
  |-- migrations
  |-- public smoke checks
  |
  v
Production deploy
  |-- migrations
  |-- public smoke checks
```

**Failure semantics:**

- Local gate failure blocks deploy.
- Missing schema or registry drift blocks deploy dry-run.
- Smoke failure means the deploy is not considered healthy even if Fly starts
  the process.
- Fly rollback remains the recovery path for bad images; DB migration rollback
  depends on the migration shape and must be reviewed per incident.

### Incident Path

```text
Component emits traces / violations / health
  |
  +--> Reeve stack-health derives current mode
  |
  +--> Chronicler assembles stories
  |       |
  |       +--> Stigmergy detects organizational/workflow patterns
  |
  +--> Vigil detects pattern anomalies
  |
  +--> Arbiter updates trust and blast-radius findings
  |
  +--> Alarms fire on declared predicates
  |
  +--> Scram evaluates kill conditions
          |
          +--> automatic action, or
          +--> Witness decision for human/two-person approval
```

**Failure semantics:**

- Missing stack mode is treated as unsafe rather than safe.
- Stale health is treated as unsafe by callers.
- Vigil never auto-rolls back; it produces investigation context.
- Scram V1 records dispatch intent but cross-stack effects are stubs until V2.
- Witness pending approval blocks non-automatic emergency action.

### Human Approval Path

```text
Caller needs judgment
  |
  v
Witness ask(decision)
  |-- persist decision
  |-- dispatch to configured surfaces
  |-- require second operator when policy says so
  |-- write rationale and output
  |
  v
Caller resumes or aborts
```

**Failure semantics:**

- No ACK from a surface triggers fallback delivery.
- Changed context invalidates second approval.
- Missing approval leaves the caller blocked or safely pending.
- Audit sink integration is the caller's responsibility in V1.

## AI-Friendly Capability Introduced

The stack is AI-friendly because it gives agents machine-readable constraints
and safe operating handles:

- **Contracts:** agents can inspect payload shapes and generate compliant data.
- **Specification artifacts:** agents can work from component maps, trust
  policies, access graphs, and executable tests rather than inferred intent.
- **Budgets:** agents cannot quietly turn one task into unbounded work.
- **Obligations:** agents can reason about data handling from a registry.
- **Trust findings:** agents can inspect access and blast-radius state before
  recommending rollout or remediation.
- **Topology:** agents can inspect service wiring instead of guessing.
- **Stories:** agents can consume bounded operational narratives rather than
  unstructured logs.
- **Adoption maps:** agents can discover missing stack artifacts before making
  invasive changes.
- **Smoke checks:** agents can verify whether a change is operationally live.
- **Human gates:** agents can request approval without inventing a workflow.
- **Authority gates:** agents can use scoped credentials and proofs rather than
  raw secrets.
- **Model routing:** agents can lower cost for repeated tasks without bypassing
  quality checks.
- **Kill switches:** agents can trigger or recommend emergency paths through
  declared mechanisms rather than shelling into production.
- **Forensics:** agents can query anomalies, traces, and audit evidence to
  explain behavior.

The important point is that these are not prompt conventions. They are
interfaces, policies, tests, and state machines that an agent can call and an
operator can audit.

## Current Maturity

### Enforced Today

- Cartographer, Constrain, and Pact exist as public stack tools for discovery,
  intent capture, and contract-first implementation.
- Aegis budget behavior is implemented in TS and Python.
- Covenant contract validation is implemented in TS and Python.
- Ledger can validate and export Reeve schemas into Baton config.
- Arbiter, Chronicler, Stigmergy, and Cartographer exist as public stack tools
  with documented responsibilities.
- Apprentice and Signet exist as public stack tools for model optimization and
  authority/credential control.
- Baton has Reeve smoke/egress config artifacts, a deployed Fly dashboard
  surface at `https://baton-stack.fly.dev/api/snapshot`, and version metadata
  at `https://baton-stack.fly.dev/v1/about`.
- Vigil has DB-backed anomaly detection and API/CLI tests.
- Witness has TS library/server and Python client tests.
- Scram has registry, evaluator, API, CLI, boot-only read-only override, and
  stubbed dispatch descriptors.
- Reeve deploy gates and public smoke endpoints are live in staging and
  production.

### Partially Enforced

- `stack-smoke` has local full-toolchain prerequisite checks, live Reeve/Baton
  endpoint checks, and the documented target flow; full end-to-end execution is
  still landing.
- Reeve still needs to migrate imports from private implementations to the
  extracted stack libraries.
- Scram's V1 dispatchers are intentional stubs pending Baton/control-plane
  endpoints.
- Witness audit integration needs to be wired into Tessera or the chosen audit
  sink.
- Vigil anomalies need to surface in Reeve's operator dashboard as a proactive
  drift signal.
- Arbiter findings, Chronicler stories, Stigmergy patterns, and Cartographer
  compatibility reports need to be represented in the stack-wide smoke and
  operator surfaces.
- Constrain/Pact/Apprentice/Signet integration points need explicit Reeve
  closeout criteria where they affect production operation.

### Not Yet Enforced

- A complete Reeve -> Baton -> Sentinel -> Tessera smoke flow.
- Real Scram action dispatch across the stack.
- Reeve operator review queue backed by Witness.
- Reeve registering Scram conditions from its TypeScript runtime.
- Full component version drift reporting across `/v1/about` endpoints.

## Closeout Checklist

These are the remaining items that determine whether the environment is merely
well-factored or fully buttoned up. The distinction matters: an implementation
phase can be staged, but a safety invariant is not optional just because the
matching failure has not happened yet. A blocker prevents the team from claiming
a fully enforced operational environment.

| Item | Status | Why it remains | Acceptance criteria |
| --- | --- | --- | --- |
| Reeve consumes extracted Aegis/Covenant | Blocker for composition | Reeve still carries private first implementations for some paths. | Reeve imports `@stack/aegis` and `@stack/covenant`; private modules are removed; existing Reeve tests pass with no behavioral drift. |
| Reeve review queue uses Witness | Blocker for human-governance composition | Witness is released, but Reeve still owns its ad-hoc review queue. | `flag_for_human_review` writes through Witness; operator inbox reads open Witness decisions; two-person policy can be configured by decision kind. |
| Reeve registers Scram conditions | Blocker for emergency composition | Scram V1 is released, but Reeve's kill-condition ownership is not wired. | Reeve-owned conditions are registered: cross-tenant lateral movement, audit-chain integrity break, and emergency read-only on PG down. |
| Scram dispatchers call real control endpoints | Blocker for emergency enforcement | Scram V1 records dispatch descriptors; cross-stack effects are explicit stubs. | Baton/control-plane endpoints exist for rollback, circuit break, tenant quarantine, global read-only, and process exit/drain; staging drill proves each path. |
| Witness audit writes to Tessera | Blocker for audit closure | Witness captures rationale, but the stack audit sink is not wired. | Every answered decision writes a Tessera-compatible audit event with decision input, operators, rationale, output, and context hash. |
| Vigil anomalies surface in Reeve | Blocker for proactive drift visibility | Vigil can be operated through CLI/API, but operators should see anomalies in their normal dashboard before drift becomes an incident. | Reeve operator dashboard has an anomalies view with filtering, details, and dismiss/escalate actions backed by Vigil. |
| Executable stack-smoke scenario | Blocker for handoff verification | Current `stack-smoke` checks local prerequisites, live Reeve smoke endpoints, live Baton snapshot, and records the target flow. | One command brings up or targets all required components and proves Reeve -> Baton -> Sentinel -> Tessera behavior end to end. |
| Component version drift reporting | Blocker for composition hygiene | Baton now exposes `/v1/about` and `stack-smoke` checks its deployed version; the rest of the components still need equivalent surfaces. | Each component exposes `/v1/about` or equivalent; `make stack-versions` reports component version and stack dependency versions, failing on unsupported drift. |
| Sentinel and Tessera decision records | Documentation debt | Both repos exist and are referenced, but their stack-level ADRs are not normalized with the new component ADR style. | Add ADR-001 files or stable design links that explain why each remains a separate stack component and what integration contract it exposes. |
| Arbiter/Chronicler/Stigmergy/Cartographer in stack-smoke | Blocker for full Exemplar coverage | These tools are part of Exemplar but were not in the first smoke scaffold. | `stack-smoke` includes at least one assertion each for Arbiter findings, Chronicler story assembly, Stigmergy pattern output, and Cartographer compatibility reporting. |
| Constrain/Pact adoption path | Blocker for gold-standard build lifecycle | Runtime tooling is documented, but the gold-standard build path should start from Constrain/Pact artifacts. | New Exemplar services have a documented path from Constrain interview -> Pact contracts/tests -> Ledger/Baton/Arbiter artifacts. |
| Apprentice routing candidates in Reeve | Blocker for measured model routing | Reeve has repeated AI tasks, but not all are evaluated for distillation. | Reeve identifies repeatable tasks eligible for Apprentice and defines quality metrics before any local-model routing. |
| Signet authority integration in Reeve | Blocker for maximal authority hygiene | Reeve has Signet-shaped stubs/boundaries, but stack-level authority policy is not fully expressed in the handoff. | Sensitive Reeve integrations use Signet-scoped credentials/proofs or a documented interim boundary with migration criteria. |

Do not remove this checklist until the acceptance criteria are satisfied. It is
the operational closeout contract for the next phase.

## Operating Guidance for Infrastructure Teams

1. **Treat docs as secondary to gates.** If a rule matters, wire it into
   pre-deploy, smoke, contract validation, or runtime state.
2. **Pin component versions.** Use release tags for stack dependencies and
   record which version each service consumes.
3. **Keep V1 stub boundaries explicit.** Do not describe Scram dispatch as real
   until Baton/control-plane endpoints are wired and rehearsed.
4. **Run smoke after migrations.** A healthy process is not the same as a
   healthy operational surface.
5. **Prefer fail-closed for safety gates.** Missing contracts, stale health, and
   unknown states should degrade capability rather than silently proceed.
6. **Practice emergency paths in staging.** Scram and Witness only become
   operationally mature after drills, not after code lands.
7. **Expose machine-readable state.** Every component should grow a `/v1/about`
   or equivalent surface that reports component version and dependency versions.

## Minimal Next Steps

The next phase should make composition real:

1. Migrate Reeve to consume `@stack/aegis` and `@stack/covenant`.
2. Back Reeve's operator review queue with Witness.
3. Wire Scram to real Baton/control-plane dispatch endpoints.
4. Feed Vigil anomalies into Reeve's operator dashboard.
5. Add Cartographer/Constrain/Pact as the expected adoption/build path for new
   Exemplar services.
6. Add Arbiter findings, Chronicler stories, Stigmergy patterns, and
   Cartographer compatibility reports to the stack-wide smoke model.
7. Identify Apprentice candidates in Reeve and document quality metrics before
   distillation.
8. Normalize Signet authority integration for sensitive Reeve actions.
9. Promote `stack-smoke` from prerequisite checks to an executable multi-service
   scenario.
10. Add component version reporting and drift checks.

When those land, the stack moves from "operationally credible primitives" to
"operationally enforced environment."
