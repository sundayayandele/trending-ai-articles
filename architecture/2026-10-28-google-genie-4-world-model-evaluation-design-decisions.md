# Architecture decisions — Evaluating Google Genie 4-style world models
**Date:** 2026-10-28  
**Status:** Proposed research/evaluation reference architecture; no Genie 4 API access or hands-on test claimed.  
**Related:** [Article](../articles/2026/10/2026-10-28-google-genie-4-world-model-evaluation.md)

## Context
DeepMind announced Genie 4 on 27 October 2026 as a trusted-tester research preview. It reports native audio, interactive scene steering and Gemini-assisted multi-step simulation, but explicitly notes limitations in physics, spatial continuity and object permanence. The following decisions concern an **independent test harness**, not Google's internal product design.

## ADR-001 — Keep world generation behind a provider interface
**Decision:** Abstract reset/step/observe behind a versioned interface; implement a deterministic simulator first and add generative providers only when authorized APIs exist.  
**Rationale:** Prevent coupling to undocumented services or unverified access.  
**Alternatives:** Hard-code a proprietary API; screen-scrape a preview UI.  
**Trade-offs:** Portability and testability improve; interface may hide provider-specific features.

## ADR-002 — Separate perceptual realism from state validity
**Decision:** Maintain explicit task state and use independent checks for physical constraints, collisions and object identity.  
**Rationale:** Realistic pixels can hide inconsistent dynamics.  
**Alternatives:** Judge quality solely by video appearance or an LLM opinion.  
**Trade-offs:** Requires ground truth or an approximate physics model; less flexible for open-ended worlds.

## ADR-003 — Deterministic evaluation first
**Decision:** Use seeded scenarios, fixed goals and reproducible control inputs before running open-ended explorations.  
**Rationale:** Baselines reveal regressions and inconsistent responses.  
**Alternatives:** Subjective demo selection.  
**Trade-offs:** Benchmarks may not capture creative value.

## ADR-004 — Keep a safety gate outside the model
**Decision:** An external controller enforces action bounds, prohibited content and timeouts; a separate evaluator decides whether episodes may be retained.  
**Rationale:** Prompt-based constraints are not sufficient enforcement.  
**Alternatives:** Model self-policing only.  
**Trade-offs:** Additional operational complexity and false rejections.

## ADR-005 — Preserve provenance and data rights
**Decision:** Record scenario prompts, provider/version, input rights, actions, output references and evaluator verdicts; preserve watermark/provenance information.  
**Rationale:** Synthetic data must remain distinguishable from real recordings.  
**Alternatives:** Unlabeled clips in a shared training corpus.  
**Trade-offs:** Storage and governance overhead.

## ADR-006 — Human review before robotics transfer
**Decision:** Use generated episodes for ideation and low-risk research first; require validated physics, supervised real-world tests and safety review before any physical deployment.  
**Rationale:** The research preview does not establish safety-critical sim-to-real reliability.  
**Alternatives:** Direct training-to-deployment pipeline.  
**Trade-offs:** Slower experimentation but fewer unsupported safety claims.

## ADR-007 — Independent benchmark and failure reporting
**Decision:** Publish denominators, success/failure cases, confidence intervals where appropriate, and known limitations.  
**Rationale:** Polished clips and selective examples are not robust evidence.  
**Alternatives:** Anecdotal comparisons.  
**Trade-offs:** Higher measurement effort and possible lower apparent scores.

## Open questions
Public developer API specification, latency and throughput, long-session consistency, pricing, export formats, physical-ground-truth fidelity, benchmark access and training-data rights remain to be confirmed.

## Sources
- https://deepmind.google/blog/genie-4-a-new-generation-of-world-simulation/
- https://gymnasium.farama.org/
- https://deepmind.google/technologies/synthid/
