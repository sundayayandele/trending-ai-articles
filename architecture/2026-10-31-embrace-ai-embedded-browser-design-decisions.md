# Architecture and Design Decisions — EMBRACE Embedded-Browser Trust Boundaries

**Date:** 2026-10-31  
**Status:** Proposed reference architecture; no implementation or exploit validation performed  
**Companion article:** [EMBRACE analysis](../articles/2026/10/2026-10-31-embrace-five-ai-coding-tools-browser-security.md)

## Scope and system context

An AI developer tool contains a browser preview, agent planner, local file tools, shell, MCP integrations and network clients. **Untrusted HTML, page scripts and local files** enter through the preview. The core failure mode is *authority confusion*: untrusted page-originated content is given privileges of the host application.

The design deliberately distinguishes **data-plane content** from **control-plane authorizations**. See [context diagram](../diagrams/2026-10-31-embrace-ai-embedded-browser-context.svg) and [deployment topology](../diagrams/2026-10-31-embrace-ai-secure-dev-deployment.svg).

## ADR-001 — Derive origin from trusted navigation metadata

**Decision:** Only the embedded browser engine/network stack may set trusted tab origin. Treat page-reported URLs as untrusted display data.

**Rationale:** Prevents spoofed location from authorizing actions.

**Alternative:** Let the page or agent report its own current URL.

**Trade-off:** More complex IPC and navigation synchronization; requires rigorous tests of redirects and history APIs.

## ADR-002 — Use a data-only context adapter

**Decision:** Transform preview content into untrusted, attributed data; never merge page text with system/developer instructions.

**Rationale:** Reduces page-to-agent instruction confusion.

**Alternative:** Pass raw page content directly to the agent as trusted instruction context.

**Trade-off:** Some agent automation becomes less seamless; a model can still be manipulated, so executor-side policy remains mandatory.

## ADR-003 — Keep tool authorization outside the model

**Decision:** A separate broker enforces per-tool scopes, explicit approvals, process allowlists and network destinations. Agent requests cannot disable or modify this policy.

**Rationale:** Model reasoning and approval prompts are not independent authorization boundaries.

**Alternative:** Ask the agent to self-police or rely solely on user clicking "Trust".

**Trade-off:** Extra engineering and user friction; approvals must display the exact proposed operation.

## ADR-004 — Constrain local-file access in every execution context

**Decision:** File access is mediated by a resource-level service enforcing canonicalized allowed roots, symlink handling and worker/frame parity.

**Rationale:** A top-level preview path check does not protect worker or helper processes.

**Alternative:** Rely on JavaScript-level path checks in the visible page.

**Trade-off:** Potential incompatibility with previews needing assets outside the workspace; allow explicit narrow grants.

## ADR-005 — Run unknown projects in ephemeral sandboxes

**Decision:** Use short-lived VMs or containers with limited mounts, no privileged host sockets and explicit egress policies.

**Rationale:** Limits blast radius when preview or agent content triggers unintended behavior.

**Alternative:** All projects and agents share a developer's fully privileged host session.

**Trade-off:** Slower startup, complex debugging and resource costs; containers with host credentials are insufficient.

## ADR-006 — Use scoped, expiring identities

**Decision:** Prefer OIDC/federation and ephemeral tokens for Git, cloud and CI; isolate production credentials from development.

**Rationale:** Reduces value of any local-file disclosure.

**Alternative:** Persistent secrets in user home and project .env files.

**Trade-off:** More identity infrastructure and periodic authentication.

## ADR-007 — Independent audit and fail-closed approvals

**Decision:** Log broker decisions, file access, origin, tool execution and egress to a tamper-resistant collector. On missing policy context, deny the operation.

**Rationale:** Post-incident analysis must not depend on the agent's self-report.

**Alternative:** Agent conversation logs only.

**Trade-off:** Privacy, retention and alert-volume considerations; protect sensitive payloads and restrict log access.

## Threat-model validation matrix

| Risk | Required test | Expected result |
| --- | --- | --- |
| Preview misreports origin | Benign mock page declares a different display URL | Trusted origin remains browser-derived |
| Untrusted page requests a shell command | Dummy command appears in page content | Agent cannot execute without broker approval |
| Worker requests a file outside project | Benign worker requests a dummy disallowed path | Access denied and audited |
| Agent attempts to widen scope | Request to disable policy or add path | Rejected independently of model |
| Unknown project requests secrets | Disposable test project requests dummy credential | No secret mounted or disclosed |
| Audit process unavailable | Simulate collector outage | High-risk action fails closed or uses defined local queue policy |

**No exploit chain or affected product was tested.** All validation proposals use harmless mock content and dummy files.

## Implementation roadmap

- **Week 1:** Inventory AI developer tools and browser previews; assess vendor updates and token scopes.
- **Weeks 2–3:** Pilot disposable dev environments, policy broker and approved file roots.
- **Weeks 4–5:** Add independent audit, egress policy, incident playbooks and approval UX.
- **Week 6:** Run benign adversarial test cases, measure false positives and developer friction, then refine.

## Sources

- Forever Security research: https://forever.security/blog/embrace-hacking-5-ai-assistants-embedded-browsers/
- Forever Security technical companion: https://forever.security/blog/embrace-hacking-5-ai-assistants-embedded-browsers-technical
- Microsoft Workspace Trust: https://code.visualstudio.com/docs/editing/workspaces/workspace-trust
- Microsoft AI trust boundaries: https://code.visualstudio.com/docs/agents/concepts/trust-and-safety
