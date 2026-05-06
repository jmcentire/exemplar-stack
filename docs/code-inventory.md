# Code Inventory and Reeve Leverage Map

Generated from the local `~/Code` workspace on 2026-05-06. This inventory is
meant to prevent the Reeve + Exemplar plan from missing load-bearing tools or
mistaking demos/research repos for production infrastructure.

This inventory uses the proactive safety stance for Reeve: safety, authority,
audit, anomaly, smoke, and routing controls are planned as guardrails before a
customer-visible failure proves the need. Implementation can phase by maturity,
but the controls below are not optional add-ons.

## Classification Rules

- **Core Exemplar toolchain:** a repo that participates in the lifecycle of an
  Exemplar system: discovery, intent capture, contract-first build, data
  obligations, routing/control, authority, observability, learning, emergency
  response, or smoke verification.
- **Reeve integration host:** code that runs as Reeve or directly bridges Reeve
  to stack components.
- **Agent/operator support:** tools that improve agent operation, review,
  memory, safety, probing, or messaging, but are not currently part of the
  Exemplar runtime contract.
- **Adjacent product/domain repo:** applications, websites, demos, or domain
  products that may use stack tools but are not themselves stack primitives.
- **Research/model repo:** papers, experiments, models, or simulations.
- **Local metadata/non-tool:** workspace configuration, shells, old folders, or
  directories without an identifiable standalone tool surface.

## Core Exemplar Toolchain

These repos should be considered first-class when planning Reeve and the
gold-standard Exemplar environment.

| Repo | Role | Reeve leverage status |
| --- | --- | --- |
| `cartographer` | Discovers existing code/backends and drafts stack artifacts. | Use for adoption scans and CI compatibility reports. |
| `constrain` | Interviews/synthesizes component maps, trust policies, Ledger hints, Pact tasks, and Baton scaffolds. | Use as the expected starting path for new Reeve/Exemplar feature families. |
| `pact` | Contract-first decomposition, executable tests, and agent implementation. | Use for new multi-component Reeve work where boundaries matter. |
| `ledger` | Data classification and obligation registry. | Already wired through Reeve `ledger-publish` and Baton egress export. |
| `arbiter` | Access auditing, consistency analysis, trust scoring, blast-radius findings. | Add to stack-smoke and Reeve operator surfaces; currently not fully represented in Reeve runtime. |
| `baton` | Circuit/adapters/routing/canaries/taint scanning. | Reeve smoke and egress configs exist; `baton-stack` is deployed for live snapshot smoke and `/v1/about` version drift checks; next step is live adapter consumption. |
| `sentinel` | PACT-key attribution and enforcement severity. | Reeve includes Sentinel observability hooks; stack-smoke needs an end-to-end attribution assertion. |
| `tessera` | Tamper-evident executable document/evidence format. | Reeve audit-chain shape exists; Witness/Scram audit writes need Tessera integration. |
| `chronicler` | Correlates logs, spans, webhooks, and incidents into stories. | Reeve has local chronicler-shaped correlation; sidecar integration remains a closeout item. |
| `stigmergy` | Detects organizational/workflow patterns from work artifacts and stories. | Feed Chronicler stories and operational findings into Stigmergy; expose relevant patterns to operators/agents. |
| `apprentice` | Distills repeated frontier-model tasks to local models under quality gates. | Reeve references Apprentice phases and has client wiring; identify eligible repeated tasks and metrics. |
| `signet` | Sovereign vault, credentials, proofs, and authority policy. | Reeve has Signet stub/client usage for credentials; normalize production authority boundaries. |
| `aegis` | TS/Python resource-budget primitive. | Reeve package references `@stack/aegis`; complete private-module migration and lint enforcement. |
| `covenant` | TS/Python contract validation and violation policy. | Reeve package references `@stack/covenant`; complete private-module migration and endpoint coverage. |
| `vigil` | Anomaly detection and forensic query over event streams. | Feed Reeve traces and surface anomalies in operator dashboard. |
| `scram` | Emergency kill-switch service. | Register Reeve conditions and replace V1 dispatch stubs with Baton/control-plane calls. |
| `witness` | HITL decisions and two-person approval. | Reeve package references `@stack/witness`; migrate operator review queue and Scram approvals. |
| `stack-smoke` | Cross-component and continuous smoke harness. | Local full-toolchain prerequisites plus live Reeve, Baton, and Baton version checks exist; next step is executable Reeve -> Baton -> Sentinel -> Tessera flow plus broader tool assertions. |
| `exemplar-stack` | Architecture, catalog, and handoff docs. | Source of truth for the integration plan and closeout criteria. |

## Reeve Integration Host

| Repo | Role | Reeve leverage status |
| --- | --- | --- |
| `reeve` | Private production integration host and operator-facing automation. | Deployed staging/production; first place to wire the full Exemplar closeout. |
| `reeve-tools` | Reeve-related website/tools scaffold. | Adjacent; not currently part of runtime integration. |

## Agent and Operator Support Tools

These are not all core Exemplar runtime components, but they are relevant to
the AI-friendly environment and should be considered when designing operator or
agent workflows.

| Repo | Role | Reeve/Exemplar relevance |
| --- | --- | --- |
| `kindex` | Persistent knowledge graph and context layer for agents. | Already used as durable memory during this work; useful for Reeve/Exemplar operating memory. |
| `Kindex-Tools` | Website/tooling wrapper for Kindex. | Presentation/support surface. |
| `simulacrum` | Adversarial framing/pushback tool. | Used to stress-test architecture and release frames; useful for ADR review. |
| `advocate` | Multi-persona code/design review. | Candidate pre-merge review support for Exemplar changes. |
| `exemplar` | AI review/pattern tool with trust scores. | Adjacent to agent review and Goodhart-style checking. |
| `agent-safe` | Small token-embedded authorization primitive. | Consider for lightweight agent capability boundaries where Signet is too heavy. |
| `signet-eval` | Policy evaluator/proxy for agent tool calls. | Consider for local agent safety gates around infrastructure operations. |
| `transmogrifier` | Register/style translation and MCP server. | Agent communication support, not core runtime. |
| `webprobe` | Mechanical/LLM web probing. | Useful for external surface audits and monitoring checks. |
| `herald` | Agent/webhook mailbox and queueing surface. | Candidate notification/agent-message substrate, not yet in Exemplar plan. |
| `Claude-code` | External Claude Code repo/fork. | Tooling dependency/reference, not Exemplar-owned. |

## Adjacent Privacy, Identity, and Messaging Repos

| Repo | Role | Exemplar relationship |
| --- | --- | --- |
| `BlindDB` | Interactive blind database/privacy demo. | Conceptually relevant to privacy posture; not in current Reeve plan. |
| `HermesP2P` | Privacy/P2P communication system. | Adjacent privacy/communication work. |
| `Signet-Tools` | Signet presentation/site tooling. | Supports Signet ecosystem. |
| `Exemplar-Tools` | Exemplar website/tooling surface. | Public presentation for the stack; not runtime. |
| `centaur.tools` | Website/tooling surface. | Adjacent presentation/product repo. |
| `stigmergicmesh-com` | Stigmergy website/presentation. | Supports Stigmergy ecosystem. |

## Adjacent Product and Domain Repos

| Repo | Role |
| --- | --- |
| `Ascend` | Organization/team performance analytics. |
| `MEA` | Managed Entry Agreement platform. |
| `talentsync-pro` | TalentSync product. |
| `SocialGame` | Empty/new local repo; no committed tool surface yet. |
| `cageandmirror` | Adjacent site/product repo; no README summary found in inventory pass. |
| `perardua` | Astro starter/site repo. |
| `herald-tools` | Local directory without git metadata found in inventory pass. |

## Research, Models, and Simulations

| Repo | Role |
| --- | --- |
| `AI` | Leap+Verify research/paper repo. |
| `leap-verify` | Regime-adaptive speculative weight prediction research. |
| `psm-model` | Pressure System Model for economic diagnostics. |
| `spectral-forecast` | Entropy-bounded time-series forecasting research/tool. |
| `speculative-forecast` | Forecasting research repo without remote found in inventory pass. |
| `drone_swarm` | Drone swarm coordination simulation/research. |
| `atlantic-convoy-sim` | Simulation repo; no git metadata found in inventory pass. |

## Local Metadata and Non-Tool Directories

| Directory | Classification |
| --- | --- |
| `.claude` | Local Claude/agent configuration. |
| `.constrain` | Local Constrain session/artifact state. |
| `.talentsync_old` | Old/local TalentSync state. |
| `mods` | Local miscellaneous directory; not classified as tool. |
| `shells` | Local shell/workspace support. |
| `etc` | Local miscellaneous support. |

## Reeve Integration Priorities

To maximally leverage the Exemplar toolchain, the Reeve plan should treat the
following as closeout criteria:

1. **Adoption/build path:** new Reeve feature families start with Cartographer
   when adopting existing code, Constrain for intent capture, and Pact for
   contract-first decomposition when multiple components are involved.
2. **Runtime hardening:** Reeve consumes `@stack/aegis`, `@stack/covenant`, and
   `@stack/witness` directly; private first implementations are removed.
3. **Data and trust:** Reeve ledger schemas publish to Ledger, Baton consumes
   Ledger-derived egress config, and Arbiter receives observed behavior for
   trust/blast-radius findings.
4. **Event learning loop:** Reeve emits traces/events into Chronicler; stories
   feed Vigil for proactive anomaly detection and Stigmergy/Apprentice for
   pattern learning and repeatable-task optimization.
5. **Authority and HITL:** Reeve sensitive actions use Signet-scoped
   credentials/proofs and Witness approval where policy requires human or
   two-person gates.
6. **Emergency control:** Reeve registers Scram conditions and Scram dispatchers
   call real Baton/control-plane endpoints.
7. **Evidence:** Witness, Scram, alarms, and operator decisions write
   Tessera-compatible audit evidence.
8. **Verification:** `stack-smoke` continuously checks local stack prerequisites
   plus live Reeve, Baton, and Baton version metadata today, then proves a real
   Reeve -> Baton -> Sentinel -> Tessera path and includes assertions for
   Arbiter, Chronicler, Stigmergy, Cartographer, Signet, Apprentice, Aegis,
   Covenant, Vigil, Scram, and Witness where each has an operational contract.

## Consistency Check

The core toolchain above is now represented in both `README.md` and
`docs/infrastructure-handoff.md`. Repos outside the core table are explicitly
classified as support, adjacent, research, or local/non-tool so they do not
silently drift into or out of the Reeve plan.
