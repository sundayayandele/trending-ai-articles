# Architecture decisions — AWS Brotli decoder remediation (CVE-2026-111497)

**Date:** 2026-10-29 | **Status:** Proposed patch playbook, not an AWS-internal architecture.  
**Article:** [AWS Brotli cross-runtime advisory](../articles/2026/10/2026-10-29-aws-brotli-lambda-patching.md)  
**Diagram:** [Remediation flow SVG](../diagrams/2026-10-29-aws-brotli-lambda-patching-context.svg)

## Context

The Node.js project fixed CVE-2026-111497 on 21 October 2026. AWS's new 28 October advisory identifies affected language runtimes and service-specific patch phases across Lambda, CodeBuild, App Runner and more. The threat is native Brotli decoding of malformed input; the cloud service control plane must not be confused with customer workloads or managed language runtimes.

## ADR-001 — Inventory runtime and native dependency separately
- **Decision:** Build a service-by-service inventory of language runtime, native Brotli package, layers, images and decoding entry points.
- **Alternatives:** Rely solely on `npm audit` or a Node version.
- **Rationale:** Brotli exposure crosses runtimes and packaging boundaries.
- **Trade-off:** Inventory work is significant; SBOM automation helps.

## ADR-002 — Follow each service's patch channel
- **Decision:** Use AWS Lambda runtime rollout guidance for managed ZIP functions; rebuild container-based functions; follow App Runner/CodeBuild/Beanstalk guidance separately.
- **Alternatives:** One global runtime-version rule.
- **Rationale:** AWS Node.js 24 has a targeted backport due to an upstream regression; rollout is region-dependent.
- **Trade-off:** More complex status tracking.

## ADR-003 — Rebuild immutable artifacts
- **Decision:** Pull patched base images, rebuild with pinned dependencies, record digest and provenance, deploy by immutable reference.
- **Alternatives:** Restart old image or reuse mutable tag.
- **Rationale:** Restart does not replace vulnerable layers.
- **Trade-off:** More build time; reproducible builds and caches help.

## ADR-004 — Progressive rollout and controlled rollback
- **Decision:** Deploy to staging, then canary/limited region, then expand; set automated error and latency gates.
- **Alternatives:** Immediate global deployment; wait for routine maintenance.
- **Rationale:** High-severity security updates need speed but can trigger regressions.
- **Trade-off:** Some temporary exposure during phased rollout; document emergency risk acceptance.

## ADR-005 — Verify runtime with service-specific evidence
- **Decision:** For Lambda ZIP runtimes inspect INIT_START patch identifiers; for containers record image digest, package/runtime inventory, and test evidence.
- **Alternatives:** Trust `process.version` alone.
- **Rationale:** AWS states Node.js 24 runtime backport is not reflected in `process.version`.
- **Trade-off:** Extra instrumentation and region-by-region verification.

## ADR-006 — Use WAF only as temporary, scoped mitigation
- **Decision:** Where immediate patching is impossible, evaluate AWS WAF Count-mode detection of inbound `Content-Encoding: br`, then Block only after impact analysis.
- **Alternatives:** Block all compressed traffic immediately; no mitigation.
- **Rationale:** May reduce one inbound attack path, but does not address upstream/queue/file decompression.
- **Trade-off:** Legitimate uploads may fail; residual exposure persists.

## ADR-007 — Patch CI and software supply chain
- **Decision:** Update pinned CodeBuild images, Dockerfiles and native packages; require SBOM/digest evidence and security review.
- **Alternatives:** Patch production only.
- **Rationale:** Vulnerable builders can reintroduce the same dependency.
- **Trade-off:** Build-system updates can disrupt pipelines; validate in a test branch.

## Operational acceptance gates

| Gate | Evidence required |
|---|---|
| Scope | Inventory of all relevant runtimes and decoding paths |
| Correct patch | AWS/vendor patch status and per-region applicability |
| Artifact | Image digest or managed runtime patch identifier |
| Functionality | Passing ordinary regression tests |
| Rollout | Canary metrics and rollback approval |
| Exceptions | Named owner, compensating control, expiration |
| Closure | Rechecked SBOM, logs, and service configuration |

## References

- AWS bulletin (28 Oct 2026): https://aws.amazon.com/security/security-bulletins/2026-155-aws/
- Node.js security release (21 Oct 2026): https://nodejs.org/en/blog/vulnerability/october-2026-security-releases
- AWS Lambda runtime updates: https://docs.aws.amazon.com/lambda/latest/dg/runtimes-update.html
