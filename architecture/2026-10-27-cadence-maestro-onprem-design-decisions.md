# Architecture and Design Decisions — Cadence Maestro On-Prem AI Engineering

**Date:** 2026-10-27  
**Status:** Conceptual proposal, pending vendor architecture and implementation validation.  
**Companion:** [Article](../articles/2026/10/2026-10-27-cadence-maestro-onprem.md)

## Scope

The target is a customer-controlled engineering environment in which an agent plans chip-design operations while deterministic EDA/SDA tools measure the results. This is a proposed reference design, **not** Cadence's published product deployment specification.

## ADR-001 — Keep proprietary design IP in a private engineering boundary

**Decision:** Locate source repositories, PDK-related materials, license services and EDA execution inside a customer-controlled network segment. Expose only task-scoped interfaces to the agent orchestrator.

**Rationale:** Sensitive design data and foundry information need explicit access control and auditable data movement.

**Alternatives:** Entirely vendor-managed SaaS (less operations, different data-transfer obligations); public-cloud private tenancy (elastic capacity but requires careful IAM/network design).

**Trade-off:** Private operation increases responsibility for capacity, upgrades, backup, physical security and disaster recovery.

## ADR-002 — Separate model reasoning from deterministic verification

**Decision:** Agents may propose changes; only approved EDA tools produce accepted engineering metrics. Keep constraints and acceptance logic outside the model's write scope.

**Alternatives:** Let an LLM judge its own outputs (fast but unreliable); prohibit agent tool use (safe but removes much of the value).

**Trade-off:** Solver time and licenses may dominate latency and cost.

## ADR-003 — Policy-controlled EDA tool gateway

**Decision:** Require allowlisted tool invocations, scoped identities, resource limits and signed action records. Use human approval for protected branches and irreversible operations.

**Alternatives:** Direct shell execution with a broad service account (simpler but unsafe); fully manual operations (slower but lower automation risk).

**Trade-off:** More integration work and approval latency; better accountability.

## ADR-004 — Model router with explicit data classification

**Decision:** Route tasks based on confidentiality, cost, context and capability. Sensitive tasks remain on approved local models. External models are available only for explicitly approved data classes.

**Alternatives:** Single model for every task (simple but inefficient); unrestricted model selection (higher leakage risk).

**Trade-off:** Routing policy and evaluation datasets must be maintained as models change.

## ADR-005 — Evaluate total accepted-change economics

**Decision:** Measure cost per **approved engineering outcome**, not just model tokens. Include inference, EDA compute, licenses, storage, engineer review and energy.

**Alternatives:** Use vendor-reported token efficiency as ROI (insufficient); count generated design suggestions (rewards quantity rather than quality).

**Trade-off:** Requires baseline instrumentation and repeated experiments.

## ADR-006 — Defense-in-depth for agent execution

**Decision:** Segment network paths, use short-lived identities, immutable run metadata, separate audit storage and explicit human sign-off. Assess NVIDIA OpenShell integration only after vendor confirmation of its actual availability and coverage.

**Alternatives:** Trust the model's prompt to follow security policy (not enforceable); deploy only a sandbox without network/identity controls (incomplete).

**Trade-off:** Stronger controls add operations and may limit experimentation.

## Acceptance criteria

- No unapproved egress of proprietary design content.
- Every EDA tool action is attributable to an approved principal and task.
- Engineering metrics include tool version, input hash and constraints.
- Protected branches cannot be changed without authorized human review.
- An agent cannot alter its own verification rules or erase audit records.
- Total cost per accepted change is compared against a reproducible baseline.

## Unresolved vendor questions

Hardware/VRAM sizing; model weights and licensing; telemetry and offline licensing; exact OpenShell integration; backup/DR support; on-prem deployment runbooks; commercial EDA agent-call entitlements.

## References

- https://www.cadence.com/en_US/home/company/newsroom/press-releases/pr/2026/cadence-introduces-maestro-a-domain-specialized-ai-model.html
- https://www.cadence.com/en_US/home/company/newsroom/press-releases/pr-ir/2026/cadence-reports-third-quarter-2026-financial-results.html
- https://openroad.readthedocs.io/en/latest/
- https://yosyshq.net/yosys/
