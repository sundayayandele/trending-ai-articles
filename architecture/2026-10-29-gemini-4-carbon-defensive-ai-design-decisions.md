# Architecture decisions — Gemini 4 Carbon defensive evaluation

**Date:** 2026-10-29  
**Status:** Proposed reference architecture; **not** a deployed or validated Google architecture.  
**Related article:** [Gemini 4 Carbon cybersecurity analysis](../articles/2026/10/2026-10-29-gemini-4-carbon-defensive-ai.md)  
**Diagram:** [Context SVG](../diagrams/2026-10-29-gemini-4-carbon-defensive-ai-context.svg)

## Context and threat model

An AI model reads potentially malicious repository content and suggests code modifications. Risks include prompt injection via code comments, fabricated test evidence, exfiltration of proprietary source, privilege escalation through tool calls, unsafe patches, and accidental merging of generated changes. **Assume model output can be wrong or adversarially influenced.**

## ADR-001 — Immutable, authorized source snapshot
- **Decision:** Analyze a commit-pinned snapshot of only approved repositories; block write access to upstream.
- **Alternatives:** Live repository clone with write token; manual pasted snippets.
- **Rationale:** Reproducibility and narrow data exposure.
- **Trade-off:** Snapshot can be stale; schedule refreshes and record provenance.

## ADR-002 — Model as adviser, never policy authority
- **Decision:** Route model findings through typed schemas, independent policy evaluation, and reviewer triage.
- **Alternatives:** Autonomous patch-and-merge; deterministic SAST only.
- **Rationale:** Benchmark performance does not confer authority to change production.
- **Trade-off:** More human review; lower blast radius.

## ADR-003 — Separate patch sandbox
- **Decision:** Apply suggested patches only in isolated disposable branches and restricted CI jobs, with default-deny outbound network and no production secrets.
- **Alternatives:** Developer laptop; shared privileged runner; microVM per task.
- **Rationale:** Generated code and test commands are untrusted.
- **Trade-off:** Isolation costs and potential test limitations; select container hardening or microVMs according to risk.

## ADR-004 — Independent verification
- **Decision:** Run deterministic unit/integration tests, SAST, dependency scanning, and review against labeled ground truth; keep raw logs.
- **Alternatives:** Trust model's self-reported results; use a single LLM judge.
- **Rationale:** Prevent evidence fabrication and test gaming.
- **Trade-off:** Curating representative test cases takes effort.

## ADR-005 — Human approval before merge or deploy
- **Decision:** Require protected branches, code owners, and approval for each patch; no direct production write permission.
- **Alternatives:** Automatic merge after CI green; staged autonomous remediation.
- **Rationale:** A passing CI run is necessary but insufficient.
- **Trade-off:** Slower response; emergency changes need separately governed break-glass procedure.

## ADR-006 — Auditable model and cost provenance
- **Decision:** Record source commit, model version, prompt template version, tool versions, token cost, findings, approvals, and CI artifact IDs in an access-controlled store.
- **Alternatives:** Store only the final PR; log entire prompts and secrets.
- **Rationale:** Incident reconstruction, comparison and governance.
- **Trade-off:** Retention/privacy burden; redact secrets and enforce lifecycle policies.

## Proposed evaluation acceptance gates

| Gate | Suggested criterion | Evidence |
|---|---|---|
| Repository authorization | All sources explicitly allowlisted | Access audit |
| Data handling | No unapproved source/secret transfer | Network and DLP logs |
| Finding accuracy | Precision and recall reported on fixed labeled corpus | Independent ground truth |
| Patch quality | All required tests pass without bypassing tests | CI artifacts and reviewer signoff |
| Governance | Every merge has a human approval | Protected branch history |
| Reproducibility | Source, prompt, model, tools pinned | Manifest and experiment log |

These are **proposed** controls, not claims about Carbon's internal implementation.

## Sources

- Google announcement (28 Oct 2026): https://blog.google/innovation-and-ai/models-and-research/gemini-models/introducing-gemini-4-carbon/
- NIST AI RMF: https://www.nist.gov/itl/ai-risk-management-framework
- OWASP GenAI security guidance: https://genai.owasp.org/llm-top-10/
