# Exemplar Stack Infrastructure Handoff

This document explains the Exemplar stack as an operational control system:
what invariants it enforces, which tools enforce them, how those tools compose,
and what capability the stack introduces for AI-assisted operations.

It is written for infrastructure engineers who need to operate, extend, or
review the stack. It is not a product overview.

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
| `baton` | `v0.3.3` | Circuit orchestration, adapter control, taint scanning, canary routing. |
| `ledger` | `v0.2.2` | Schema registry and data obligation manager. |
| `stack-smoke` | `v0.1.0` | Cross-component smoke harness scaffold. |
| `exemplar-stack` | `v0.1.0+` | Architecture map and coordination docs. |
| `reeve` | private, deployed | First production integration host. |

Reeve is currently deployed to staging and production with live smoke checks for
liveness, readiness, audit-chain shape, stack mode, and registry population.

## Component Responsibilities

The stack has two classes of components: operational primitives and integration
hosts.

| Component | Responsibility | Operational boundary |
| --- | --- | --- |
| Reeve | Operator-facing automation and first full integration host. | Owns business workflows, route handlers, operator UI, and first production proof. |
| Baton | Circuit and adapter control plane. | Owns traffic routing, health, canary routing, taint scanning, and egress config consumption. |
| Ledger | Schema and obligation registry. | Owns field classifications, retention/masking/export obligations, and downstream config generation. |
| Sentinel | Attribution and severity enforcement. | Owns PACT-key attribution and enforcement severity for data movement. |
| Tessera | Audit evidence and hash-chain integrity. | Owns append-only evidence and integrity checks for operational actions. |
| Aegis | Resource-budget primitive. | Owns timeout/fallback semantics around blocking calls. |
| Covenant | Contract validation primitive. | Owns runtime shape validation and violation policy. |
| Vigil | Pattern-divergence detector. | Owns anomaly detection and forensic query over event streams. |
| Scram | Emergency kill-switch primitive. | Owns kill-condition evaluation and emergency action authority. |
| Witness | Human decision primitive. | Owns approval, rationale, and two-person coordination. |
| Stack Smoke | Cross-component smoke harness. | Owns assertions that prove multiple components work together. |

The important boundary is that Reeve uses these controls; it should not be the
permanent owner of them. Reeve was the proving ground. The extracted repos are
the reusable stack surface.

## The Operational Invariants

### 1. Blocking Work Must Be Budgeted

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

### 2. Cross-Boundary Data Must Match a Contract

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

### 3. Data Obligations Must Be Declared Once and Propagated

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

### 4. Egress and Service Topology Must Be Controllable

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

### 5. Routine Health Must Be Observable and Fail Closed

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

### 6. Pattern Drift Must Be Detectable Below Alarm Thresholds

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

### 7. Catastrophic Actions Need an Emergency Authority Path

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

### 8. Human Judgment Must Be a First-Class Primitive

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

### 9. Cross-Component Behavior Must Be Tested as a Flow

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
Operator or inbound event
  |
  v
Reeve
  |-- wraps external calls through aegis
  |-- validates egress and ingress through covenant
  |-- consults Ledger-derived obligations for data handling
  |-- emits operational traces and audit evidence
  |
  v
Baton adapters
  |-- route or pause traffic
  |-- apply egress masking from Ledger
  |-- scan for taint fingerprints
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
  +--> Vigil detects pattern anomalies
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
- **Budgets:** agents cannot quietly turn one task into unbounded work.
- **Obligations:** agents can reason about data handling from a registry.
- **Topology:** agents can inspect service wiring instead of guessing.
- **Smoke checks:** agents can verify whether a change is operationally live.
- **Human gates:** agents can request approval without inventing a workflow.
- **Kill switches:** agents can trigger or recommend emergency paths through
  declared mechanisms rather than shelling into production.
- **Forensics:** agents can query anomalies, traces, and audit evidence to
  explain behavior.

The important point is that these are not prompt conventions. They are
interfaces, policies, tests, and state machines that an agent can call and an
operator can audit.

## Current Maturity

### Enforced Today

- Aegis budget behavior is implemented in TS and Python.
- Covenant contract validation is implemented in TS and Python.
- Ledger can validate and export Reeve schemas into Baton config.
- Baton has Reeve smoke/egress config artifacts.
- Vigil has DB-backed anomaly detection and API/CLI tests.
- Witness has TS library/server and Python client tests.
- Scram has registry, evaluator, API, CLI, boot-only read-only override, and
  stubbed dispatch descriptors.
- Reeve deploy gates and public smoke endpoints are live in staging and
  production.

### Partially Enforced

- `stack-smoke` is still a scaffold for full end-to-end execution.
- Reeve still needs to migrate imports from private implementations to the
  extracted stack libraries.
- Scram's V1 dispatchers are intentional stubs pending Baton/control-plane
  endpoints.
- Witness audit integration needs to be wired into Tessera or the chosen audit
  sink.
- Vigil anomalies need to surface in Reeve's operator dashboard.

### Not Yet Enforced

- A complete Reeve -> Baton -> Sentinel -> Tessera smoke flow.
- Real Scram action dispatch across the stack.
- Reeve operator review queue backed by Witness.
- Reeve registering Scram conditions from its TypeScript runtime.
- Full component version drift reporting across `/v1/about` endpoints.

## Closeout Checklist

These are the remaining items that determine whether the environment is merely
well-factored or fully buttoned up. The distinction matters: a deferred item is
known and non-blocking for current operation; a blocker prevents the team from
claiming a fully enforced operational environment.

| Item | Status | Why it remains | Acceptance criteria |
| --- | --- | --- | --- |
| Reeve consumes extracted Aegis/Covenant | Blocker for composition | Reeve still carries private first implementations for some paths. | Reeve imports `@stack/aegis` and `@stack/covenant`; private modules are removed; existing Reeve tests pass with no behavioral drift. |
| Reeve review queue uses Witness | Blocker for human-governance composition | Witness is released, but Reeve still owns its ad-hoc review queue. | `flag_for_human_review` writes through Witness; operator inbox reads open Witness decisions; two-person policy can be configured by decision kind. |
| Reeve registers Scram conditions | Blocker for emergency composition | Scram V1 is released, but Reeve's kill-condition ownership is not wired. | Reeve-owned conditions are registered: cross-tenant lateral movement, audit-chain integrity break, and emergency read-only on PG down. |
| Scram dispatchers call real control endpoints | Blocker for emergency enforcement | Scram V1 records dispatch descriptors; cross-stack effects are explicit stubs. | Baton/control-plane endpoints exist for rollback, circuit break, tenant quarantine, global read-only, and process exit/drain; staging drill proves each path. |
| Witness audit writes to Tessera | Blocker for audit closure | Witness captures rationale, but the stack audit sink is not wired. | Every answered decision writes a Tessera-compatible audit event with decision input, operators, rationale, output, and context hash. |
| Vigil anomalies surface in Reeve | Deferred operational UX | Vigil can be operated through CLI/API, but operators should see anomalies in their normal dashboard. | Reeve operator dashboard has an anomalies view with filtering, details, and dismiss/escalate actions backed by Vigil. |
| Executable stack-smoke scenario | Blocker for handoff verification | Current `stack-smoke` checks prerequisites and records the target flow. | One command brings up or targets all required components and proves Reeve -> Baton -> Sentinel -> Tessera behavior end to end. |
| Component version drift reporting | Deferred operational maturity | Released repos exist, but there is no central drift report yet. | Each component exposes `/v1/about` or equivalent; `make stack-versions` reports component version and stack dependency versions, failing on unsupported drift. |
| Sentinel and Tessera decision records | Documentation debt | Both repos exist and are referenced, but their stack-level ADRs are not normalized with the new component ADR style. | Add ADR-001 files or stable design links that explain why each remains a separate stack component and what integration contract it exposes. |

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
5. Promote `stack-smoke` from prerequisite checks to an executable multi-service
   scenario.
6. Add component version reporting and drift checks.

When those land, the stack moves from "operationally credible primitives" to
"operationally enforced environment."
