# Architecture and Design Decisions — Cursor/Claude Code Extension Supply-Chain Incident

**Date:** 2026-10-31  
**Status:** Proposed reference design; not a tested deployment  
**Companion article:** [Claude Code extension incident](../articles/2026/10/2026-10-31-cursor-claude-code-extension-backdoor-response.md)

## System context and trust boundaries

A developer workstation runs IDE extensions that may execute code with the user's privileges. It often connects to source-control providers, CI, package registries and cloud services. The main trust boundary is **untrusted extension bytes entering a credential-bearing workstation**. A second boundary is the workstation's access to remote administrative systems.

See [context SVG](../diagrams/2026-10-31-cursor-claude-extension-context.svg) and [incident-response flow SVG](../diagrams/2026-10-31-cursor-claude-extension-response-flow.svg). Diagrams depict a *recommended* enterprise design, not Cursor's implementation.

## ADR-001 — Enforce an approved extension distribution policy

**Decision:** Maintain allowlisted extension IDs, publishers, approved channels and version/digest inventory. Require change review for high-privilege extensions.

**Rationale:** A trusted display name or publisher identity cannot alone prove artifact integrity.

**Alternative:** Unrestricted marketplace auto-updates.

**Trade-off:** Version pinning delays security fixes; combine staged rollout with urgent exceptions and rapid provenance verification.

## ADR-002 — Use non-administrator developer sessions

**Decision:** Editors and extensions run under standard user accounts; elevation requires separate just-in-time approval.

**Rationale:** Reduces the ability to install services or system-wide persistence, although user-level persistence and data access remain possible.

**Alternative:** Developer local administrator by default.

**Trade-off:** More support overhead for local tools and drivers.

## ADR-003 — Remove standing production secrets from laptops

**Decision:** Broker access through short-lived identity federation and scoped credentials, preferably with separate high-trust deployment runners.

**Rationale:** Limits credential theft and post-compromise blast radius.

**Alternative:** Long-lived API keys in .env files, SSH agents and local credential stores.

**Trade-off:** Requires identity infrastructure and occasional reauthentication.

## ADR-004 — Separate risky projects and extension evaluation

**Decision:** Use disposable, constrained VMs or dev environments for untrusted code and extensions, with minimal host mounts and egress.

**Rationale:** Reduces the impact of code running with editor privileges.

**Alternative:** Run every project in the same privileged host IDE.

**Trade-off:** Extra latency, GPU/tooling complexity and developer friction; host-socket passthrough can negate isolation.

## ADR-005 — Correlate endpoint and SaaS telemetry

**Decision:** Monitor extension changes, authorized_keys modifications, persistence artifacts, outbound tunnels and downstream cloud/Git activity in a central security pipeline.

**Rationale:** Single-file indicators miss variants and cannot prove a negative.

**Alternative:** Manual endpoint checks only after public advisories.

**Trade-off:** Endpoint privacy, alert noise, storage costs and SOC triage effort; restrict access and define retention.

## ADR-006 — Adopt rebuild and credential-rotation gates

**Decision:** Confirmed developer endpoint compromise triggers evidence preservation, containment, token/session revocation, controlled rebuild and re-enrollment checks.

**Rationale:** Uninstalling the original package may leave independently running persistence.

**Alternative:** Remove extension and resume work.

**Trade-off:** Developer downtime and potentially disruptive credential rotations; pre-plan emergency workflows.

## Threat model and evaluation

| Threat | Preventive control | Detective control | Recovery criterion |
| --- | --- | --- | --- |
| Modified extension served under known ID | Approved channel + digest review | Unexpected package hash or install event | Verified reinstall from trusted channel |
| User-level SSH-key persistence | Non-admin host + monitored authorized_keys | New unauthorized key | Key removed, host validated or rebuilt |
| Reverse tunnel attempt | Outbound policy + EDR | Unexpected SSH process/network pattern | Process removed and no persistence |
| Cloud/Git credential misuse | Federated short-lived scoped identity | Unusual API activity | Tokens/sessions revoked, audit reviewed |
| Incomplete cleanup | Standardized response playbook | Post-remediation telemetry | Clean rebuild / documented validation |

### Implementation and verification gates

1. Pilot extension allowlisting with one engineering team; measure false positives and update latency.
2. Test benign detection artifacts on a disposable VM, **not** the real malicious extension.
3. Confirm incident ticket contains affected distribution channel, time window, host inventory, evidence, rotation records and recovery approval.
4. Exercise a simulated revocation and rebuild quarterly.
5. Re-evaluate the design after vendor incident postmortems.

## Sources

- Cursor security notice: https://prod.cursor.com/blog/claude-code-extension-security-notice
- VS Code Workspace Trust: https://code.visualstudio.com/docs/editing/workspaces/workspace-trust
- VS Code AI security: https://code.visualstudio.com/docs/agents/run/security
- GitHub secure-use guidance: https://docs.github.com/en/code-security/tutorials/secure-your-organization/protect-against-threats
