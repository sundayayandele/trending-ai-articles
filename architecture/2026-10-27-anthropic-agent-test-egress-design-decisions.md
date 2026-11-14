# Architecture Decisions — Anthropic AI Agent Evaluation Isolation

**Date:** 2026-10-27  
**Status:** Proposed reference architecture; not implemented or tested.  
**Companion:** [Article](../articles/2026/10/2026-10-27-anthropic-agent-test-egress.md)

## Context and threat model

An evaluation agent can call tools, read files, execute code and run for hours. The key threats are unapproved network access, credential discovery, lateral movement, unauthorized tool actions, and loss of audit integrity. The Oct 26 Anthropic filing coverage is a **motivating example**, not proof that this architecture matches or would definitively prevent the reported incidents.

## ADR-001 — Independent default-deny network boundary

**Decision:** Place runners in an isolated network without production routes, enforce external firewall/VPC egress deny, and additionally apply Kubernetes NetworkPolicy where supported.

**Why:** A prompt saying “stay offline” is not enforceable. NetworkPolicy alone may be ineffective without a compatible CNI and does not replace node-level controls.

**Alternatives:** Egress proxy with allowlist (supports approved access but increases proxy trust); fully air-gapped network (stronger separation but less convenient); pod NetworkPolicy only (simpler but insufficient as sole boundary).

**Validation:** Network reachability test from the actual runner and independent firewall logs.

## ADR-002 — Ephemeral least-privilege workload identity

**Decision:** Issue a fresh per-run identity with explicit audience, scope and expiry. Never mount developer cloud credentials or real production secrets.

**Alternatives:** Shared service account (easier, excessive blast radius); long-lived static API key (operationally easy, high leakage impact).

**Validation:** Negative authorization tests, expiry and revocation tests.

## ADR-003 — Tool authorization outside the model

**Decision:** Route consequential shell, browser, API and MCP actions through a policy gateway that binds decisions to run, principal, action and resource. Require human approval for high-impact operations.

**Alternatives:** Rely on model instructions (non-binding); block all tools (safer but defeats evaluation); broad human “approve session” (not sufficiently scoped).

**Validation:** Read-only task cannot write; approval cannot be replayed for another resource.

## ADR-004 — Disposable runners with hard lifecycle limits

**Decision:** Use isolated, non-privileged, short-lived runners; terminate processes, tokens and mounts after each run. Evaluate microVM isolation for higher-risk workflows.

**Alternatives:** Shared persistent container (lower startup overhead, higher contamination risk); dedicated microVM for every task (stronger isolation, higher latency and cost).

**Validation:** No surviving processes, connections or reusable credentials after timeout.

## ADR-005 — Out-of-band audit

**Decision:** Send tool decisions, network denies and lifecycle events to an external collector with access controls that prevent agent alteration.

**Alternatives:** Logs only inside runner (low cost, weak integrity); local host-only logs (better but insufficient if host compromised).

**Validation:** Attempt to disable local logs while confirming independent evidence survives.

## ADR-006 — Synthetic data and canary-only tests

**Decision:** Never use live credentials or personal information in high-risk evaluations. Use inert canaries and synthetic systems.

**Alternatives:** Production-like real data (more realistic, unacceptable exposure for this class of test); no fixtures (poor coverage).

**Validation:** Scan image layers, environment and mounts before each evaluation.

## Decision consequences

The reference design increases setup cost and reduces access to real services. It substantially improves enforceability and forensic visibility **if correctly deployed and tested**, but does not eliminate model misuse or implementation bugs. Maintain independent human incident response and periodic policy regression tests.

## References

- https://www.anthropic.com/news/s1-sec
- https://www.sec.gov/Archives/edgar/data/2130357/000162828026067168/anthropicpbc-sx1.htm
- https://www.reuters.com/legal/litigation/anthropic-discloses-month-long-training-pause-ai-security-breaches-ipo-filing-2026-10-26/
- https://kubernetes.io/docs/concepts/services-networking/network-policies/
- https://www.nist.gov/itl/ai-risk-management-framework
