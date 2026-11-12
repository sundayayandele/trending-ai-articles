# Architecture and Design Decisions: VOLTA AI Infostealer Defender Playbook

- **Date:** 2026-11-01
- **Status:** Proposed reference architecture; **not production tested**
- **Related article:** [AI Malware Now Ranks Its Victims](../articles/2026/11/2026-11-01-volta-ai-infostealer-defender-playbook.md)
- **Diagram:** [Defensive SOC architecture](../diagrams/2026-11-01-volta-ai-infostealer-defender-playbook-defensive-architecture.svg)
- **Primary research:** https://blog.talosintelligence.com/appraisal-by-ai-model-driven-victim-prioritization-in-the-volta-infostealer/

## Context and boundaries

Cisco Talos reported on 31 October 2026 that the VOLTA infostealer combines conventional credential and session theft with an optional LLM-driven victim appraisal stage. The incident-response design must protect endpoints, identities and applications regardless of whether the malware successfully contacts a model. It must not assume a specific deployment prevalence or that all variants behave identically.

**Trust boundaries:** managed endpoints; security telemetry ingestion; identity/SaaS/cloud APIs; privileged response plane; evidence storage; optional analyst-facing AI summarization. Compromised endpoints and all collected logs are untrusted inputs.

## ADR-001 — Correlate endpoint and identity signals

- **Decision:** Ingest EDR process, file and network telemetry alongside IdP sign-ins, SaaS sessions, cloud API audit and secret-use events into a SIEM.
- **Rationale:** A self-deleting infostealer can leave identity misuse after the binary disappears; API calls alone are insufficient.
- **Alternatives:** Endpoint-only antivirus; model-API domain blocking.
- **Trade-offs:** More telemetry cost and tuning effort, but better coverage across stolen tokens and services.
- **Verification:** Tabletop cases must link suspicious endpoint activity to a follow-on identity event.

## ADR-002 — Prioritize behavioral hunting over a single hash

- **Decision:** Use Talos's sample SHA-256 for enrichment, while hunting suspicious headless Chrome launches and credential-store access with parent-process context.
- **Rationale:** A sample hash is brittle and headless browsing can be legitimate.
- **Alternatives:** Hash-only IOC matching; blanket headless Chrome bans.
- **Trade-offs:** Behavioral rules need baselines and can generate false positives.
- **Verification:** Include both legitimate browser testing and suspicious synthetic process trees in detection tests.
- **Reference:** https://attack.mitre.org/detectionstrategies/DET0037/

## ADR-003 — Separate endpoint containment from credential containment

- **Decision:** Treat isolation, session revocation, password resets, SSH/cloud key rotation and app-specific session invalidation as distinct response actions.
- **Rationale:** An isolated workstation cannot undo already-stolen tokens; identity-provider revocation may not terminate an application-issued cookie.
- **Alternatives:** Reboot/reimage only; password reset only.
- **Trade-offs:** Coordinating several providers increases response effort but addresses independent attack surfaces.
- **Verification:** Confirm session invalidation at each affected provider, then audit for reuse.
- **Reference:** https://learn.microsoft.com/en-us/entra/identity/users/users-revoke-access

## ADR-004 — Human approval for destructive or disruptive response

- **Decision:** Require an authorized incident commander to approve endpoint isolation, credential resets and key revocation except for separately approved emergency policies.
- **Rationale:** These actions can interrupt production and cause loss of forensic access.
- **Alternatives:** Fully autonomous LLM remediation; entirely manual untracked actions.
- **Trade-offs:** Approval adds latency; playbooks, clear escalation paths and preapproved emergency cases reduce delay.
- **Verification:** Every privileged response action records approver, reason, time, target and outcome.

## ADR-005 — Protect evidence and secrets

- **Decision:** Store collected artifacts in an access-controlled evidence vault, encrypt at rest/in transit, minimize retention and redact secrets from analyst summaries.
- **Rationale:** VOLTA debug logs and account traces can themselves contain sensitive material.
- **Alternatives:** Copy logs to shared chat; paste into public AI assistants.
- **Trade-offs:** Redaction and chain-of-custody workflows add operational overhead but reduce secondary exposure.
- **Verification:** Test role-based access and audit logs; use synthetic fixtures.

## ADR-006 — Optional AI is advisory and isolated

- **Decision:** If an AI summarizer is used, feed it only sanitized, non-executable evidence; provide no production credentials or direct response privileges.
- **Rationale:** Incident artifacts are untrusted and may contain prompt-injection text. Model output is not an authoritative signal.
- **Alternatives:** Give the model access to EDR/IdP admin tokens; avoid AI entirely.
- **Trade-offs:** Less automation but clearer accountability. Avoiding AI entirely is a valid deployment choice.
- **Verification:** Prompt-injection and data-leak tests; review model summaries against source evidence.

## ADR-007 — Design for measurable false positives and operational safety

- **Decision:** Begin with read-only hunting and synthetic tabletop data; add alerts only after tuning with legitimate headless-browser activity.
- **Rationale:** Headless Chrome is not malware by itself.
- **Alternatives:** Auto-isolate every endpoint launching Chrome headlessly.
- **Trade-offs:** Slower rollout, fewer unnecessary outages.
- **Verification:** Measure precision, triage time, false-positive rate and missed synthetic cases.

## Threat model

| Threat | Asset | Defensive boundary | Residual risk |
|---|---|---|---|
| Infostealer collects cookies | Browser and SaaS sessions | Endpoint controls + app session revocation | Unobserved sessions on external services |
| Stolen cloud key reused | Cloud workloads | Short-lived credentials + key rotation + audit | Delayed detection of unauthorized actions |
| Malicious log text manipulates analyst AI | SOC workflows | Sanitization + no privileged AI actions | Human overreliance on summaries |
| IOC changes | Detection pipeline | Behavioral correlation + threat-intel updates | Novel variants and telemetry gaps |
| Excessive response privileges | Production systems | RBAC + approvals + audit | Emergency workflow misuse |

## Rollout and rollback

**Phase 1:** Confirm required telemetry, validate read-only KQL, baseline developer browser automation and identify affected SaaS session controls.

**Phase 2:** Implement documented response playbooks and evidence handling; conduct tabletop exercises.

**Phase 3:** Pilot alerting with manual review and false-positive tracking; only then consider automated *low-impact* enrichment.

**Rollback:** Disable new alert rules if they create unacceptable false positives; release isolation through authorized EDR procedures after containment; rotate any compromised automation credentials. Do not automatically reverse legitimate credential revocation without a fresh trust assessment.

## Sources

- Cisco Talos (2026-10-31): https://blog.talosintelligence.com/appraisal-by-ai-model-driven-victim-prioritization-in-the-volta-infostealer/
- Microsoft Defender isolation: https://learn.microsoft.com/en-us/defender-endpoint/api/isolate-machine
- Microsoft Defender hunting schema: https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-deviceprocessevents-table
- Microsoft Entra emergency revocation: https://learn.microsoft.com/en-us/entra/identity/users/users-revoke-access
- MITRE ATT&CK browser credential detection: https://attack.mitre.org/detectionstrategies/DET0037/
