# Architecture decisions — Oracle/OpenAI AI Security Mesh analysis
**Date:** 2026-10-28  
**Status:** Proposed reference architecture; not Oracle implementation documentation.  
**Related article:** [Full analysis](../articles/2026/10/2026-10-28-oracle-openai-security-mesh.md)

## Context and assumptions
The announced managed service uses OpenAI cybersecurity models and Oracle security orchestration. Exact internal topology, API contracts, cost and availability are not publicly validated here. These decisions describe a **customer-side defensive evaluation pattern** and should be reviewed against Oracle's actual documentation before procurement.

## ADR-001: Explicit, expiring scope authorization
**Decision:** Maintain a signed allowlist of target assets, allowed methods, owners, test windows and exclusions. Reject undeclared targets at the execution gateway.  
**Rationale:** Natural-language instructions are not reliable access-control boundaries.  
**Alternatives:** Broad role-based permission; verbal authorization.  
**Consequences:** Stronger auditability; administrative overhead to maintain scope.

## ADR-002: Independent agent execution boundary
**Decision:** Use isolated, disposable runners with per-task identities, egress controls, resource limits and no default production write permissions.  
**Alternatives:** Shared worker container, privileged production service account.  
**Rationale:** Generated tool calls and retrieved content can be untrusted.  
**Consequences:** Extra infrastructure cost and potentially less test fidelity.

## ADR-003: Separate validation from finding generation
**Decision:** A second deterministic runner or independently configured validator reproduces findings before severity assignment.  
**Alternatives:** Trust a single agent's assertion; manual-only triage.  
**Rationale:** Reduces hallucinated and non-reproducible reports.  
**Consequences:** More compute and analyst review; does not guarantee zero false positives.

## ADR-004: Human approval before code or infrastructure mutation
**Decision:** Convert remediation suggestions into PRs with CI checks and an independently verified human approval.  
**Alternatives:** Direct automated patching; alert-only scanning.  
**Rationale:** Prevents an agent from unilaterally changing production.  
**Consequences:** Slower remediation but stronger change control and rollback.

## ADR-005: Defense in depth for network access
**Decision:** On OCI, evaluate ZPR together with routes, NSGs and security lists; elsewhere use equivalent native enforcement. Verify attributes and policies before attaching them to production resources.  
**Alternatives:** Agent prompt instructions; perimeter firewall alone.  
**Rationale:** The enforcement boundary must be outside the agent.  
**Consequences:** Cross-cloud policies differ; increased operational complexity.

## ADR-006: Out-of-band telemetry and emergency stop
**Decision:** Send audit evidence to a separately controlled log plane and provide an operator kill switch that revokes credentials and blocks runner egress.  
**Alternatives:** Logs stored only inside the agent workspace.  
**Rationale:** Preserve independent evidence if the agent malfunctions or is compromised.  
**Consequences:** Additional storage, alert tuning and incident rehearsal.

## ADR-007: Prove value with a controlled defect corpus
**Decision:** Pilot in staging with synthetic cross-tenant authorization cases, baseline tests and measured false positives.  
**Alternatives:** Immediate production rollout; trust vendor claims.  
**Rationale:** Separate product capability from marketing assertions.  
**Consequences:** Slower procurement, but reproducible acceptance evidence.

## Open questions
Exact Oracle API/connector support, deployment and data residency, cost model, SOC integrations, SLAs, and independent benchmark results remain to be verified with Oracle.

## References
- https://www.oracle.com/fr/news/announcement/ai-world-oracle-advances-enterprise-security-for-the-ai-era-2026-10-27/
- https://www.oracle.com/artificial-intelligence/security-mesh/
- https://blogs.oracle.com/ai-and-datascience/introducing-oracle-ai-security-mesh
- https://docs.oracle.com/en-us/iaas/Content/zero-trust-packet-routing/overview.htm
