---
title: "Claude Code Extension Backdoored in Cursor: How to Check Your Developer Laptop and Contain the Risk"
date: 2026-10-31
last_verified: 2026-10-31
description: "An evidence-based response guide to the October 29, 2026 Cursor marketplace Claude Code extension compromise, with macOS and Windows triage, incident-response architecture and prevention controls."
tags: [Cursor, Claude Code, developer security, software supply chain, incident response, VS Code extensions, SSH, DevSecOps]
keywords: ["Cursor Claude Code extension compromised", "Claude Code 2.1.313", "Cursor marketplace security notice", "svc-telemetry SSH key", "developer workstation incident response"]
status: draft-for-editorial-review
canonical_url: null
---

# Claude Code Extension Backdoored in Cursor: How to Check Your Developer Laptop and Contain the Risk

> **Security advisory and publication check (31 October 2026):** This article is based on Cursor's 29 October notice, updated 30 October at 11:30 a.m. Pacific. Check the live vendor advisory before publishing or acting. The confirmed affected distribution channel is **Cursor's extension marketplace**, not all Claude Code installations. Do not infer that Anthropic's own build or other marketplaces were compromised.

## Executive summary

On **29 October 2026**, a maliciously modified copy of the **Anthropic.claude-code** editor extension was available through Cursor's VS Code Extension Marketplace for approximately **11 hours**, from **2:27 a.m. to 1:35 p.m. Pacific**. Cursor's notice identifies affected **macOS and Windows marketplace packages labelled 2.1.312 and 2.1.313**. It says the malicious packages were **not developed by Anthropic** and that it has no evidence that other versions or official distribution channels were affected [1].

The payload collected system metadata and attempted to add an attacker-controlled SSH key, enable SSH, create a reverse tunnel, and install persistence. Cursor says the tunnel-enablement step failed on most non-administrator macOS and Windows sessions, but **the SSH key, persistence and host-data reporting could still occur** [1]. The attacker's server was taken offline, but that does not reverse local changes. Uninstalling the extension alone may leave persistence behind.

Reuters and WIRED reported that **up to 5,000 users may have been exposed** [2][3]. This is an upper bound on potential exposure, **not** evidence that 5,000 hosts were successfully accessed or that secrets were stolen. The initial publication route remains under investigation. Anthropic staff said the modified extension was not built or published by Anthropic and that Anthropic systems were not compromised [3].

**Action for teams:** If developers installed or auto-updated the named extension from Cursor's marketplace in the affected window, inventory those endpoints, use Cursor's official checks and cleanup guidance, preserve relevant evidence, isolate confirmed cases, rotate accessible credentials from a known-clean device, and consider a rebuild.

## Why this is a distinct YouTube opportunity

This is a new **supply-chain incident**, not a generic AI-model announcement. Developers increasingly run editors with Git, cloud credentials, SSH keys, package registries, deployment tokens and access to production. The practical story is the difference between an extension's *brand identity* and its *verified artifact provenance*.

A high-utility video can show **read-only triage**, a safe decision tree and enterprise prevention controls. News interest is independently supported by Cursor's notice and subsequent Reuters/WIRED reporting; **YouTube demand and competitor counts have not been measured**. Do not describe the topic as proven low-competition.

## Confirmed timeline (all times Pacific)

| Date/time | Verified event |
| --- | --- |
| 29 Oct, ~02:27 | Start of Cursor marketplace exposure window, per Cursor. |
| 29 Oct, ~13:35 | End of the exposure window; Cursor removed the modified packages. |
| 29 Oct | Cursor temporarily disabled its marketplace, investigated other extensions, rotated keys and rebuilt the marketplace. |
| 30 Oct, 11:30 | Cursor updated indicators of compromise and clarified affected versions. |
| 31 Oct | This analysis was checked against the current vendor notice. |

**Scope caveat:** Cursor's advisory names **2.1.312 and 2.1.313**, but only specific *modified artifacts served from Cursor's marketplace during the incident* are implicated. A version string alone is not proof of infection, and a newer installed version alone is not proof that the host was cleaned.

## What the malicious extension attempted

The extension's modified code ran as part of extension loading. Cursor describes four effects:

1. **Host reconnaissance:** hostname, username, operating system, home and working directories, Git email, public IP and **environment variable names**. The vendor's initial disclosure does not prove that environment-variable values were collected at this stage.
2. **Persistence preparation:** addition of an attacker-controlled public key labelled **svc-telemetry** to SSH authorized keys.
3. **Remote access attempt:** an attempt to enable an SSH server and establish an outbound reverse tunnel. On most normal macOS/Windows sessions, required administrator privileges and errors prevented successful SSH activation.
4. **Repeated execution:** macOS LaunchAgent or Windows scheduled task / Startup fallback, with background retries.

The vendor lists exact file paths, hashes and network indicators in its live advisory [1]. Treat them as a current authoritative source rather than relying on copied IOCs that may be corrected later.

### What has *not* been established

- The exact entry point by which the malicious artifact reached the marketplace.
- That all users of Cursor, Claude Code, VS Code or Open VSX were affected.
- That every exposed host established an SSH session or lost secrets.
- That this incident caused any unrelated exchange theft or cloud breach.
- That updating or uninstalling an extension automatically removes attacker persistence.

## Safe, read-only triage for individual developers

**Before running commands:** Confirm whether you used the **Cursor marketplace** during the exposure window. If this is a corporate machine, contact your incident-response team before modifying evidence. The following examples are *read-only indicators*, not definitive all-clear tests.

### macOS: inspect the named SSH-key marker

~~~bash
if [ -f "$HOME/.ssh/authorized_keys" ]; then
  grep -n 'svc-telemetry' "$HOME/.ssh/authorized_keys" || true
else
  printf 'No user authorized_keys file found\n'
fi
~~~

An empty result **does not establish safety**: inspect extension-installation telemetry and persistence evidence. Cursor also lists **~/Library/LaunchAgents/com.apple.sysmond.plist** and files in **~/.config/** among its indicators [1].

### Windows: inspect SSH authorized keys without modifying them

~~~powershell
$paths = @(
  "$HOME\.ssh\authorized_keys",
  "C:\ProgramData\ssh\administrators_authorized_keys"
)
foreach ($path in $paths) {
  if (Test-Path -LiteralPath $path) {
    Select-String -Path $path -SimpleMatch 'svc-telemetry'
  }
}
~~~

Review the **SysMon** scheduled-task name and **sysmon.cmd** Startup artifact only in the context of Cursor's advisory. Names alone may collide with legitimate software; validate full paths, command lines and timestamps.

### Enterprise triage sequence

1. **Scope:** Query endpoint-management and EDR records for Cursor users and extension installation/update events during the affected window.
2. **Confirm:** Compare extension provenance, package hashes and vendor IOCs; check the SSH-key marker and persistence artifacts.
3. **Contain:** Isolate confirmed or strongly suspected hosts from sensitive networks, retaining access for responders.
4. **Preserve:** Export relevant logs, task/LaunchAgent definitions, extension metadata and hashes before cleanup where policy requires.
5. **Remediate:** Use Cursor's *officially hosted* macOS/Windows remediation scripts only after reviewing the source and following internal change controls. Do not copy unknown scripts from social posts.
6. **Rotate:** From a clean device, revoke and replace reachable SSH credentials, cloud keys, Git tokens, package-publishing tokens, CI/CD secrets and active sessions according to the host's access.
7. **Rebuild and validate:** Consider reimaging for high-trust developer machines. Confirm absence of persistence, re-enroll EDR and restore only verified extensions.
8. **Monitor:** Hunt for related activity in cloud audit trails, Git provider logs, SSH access records and deployment events.

**Important:** The absence of the key marker is only one check. Do not use a single negative test to close the incident. Do not run destructive cleanup until evidence needs have been considered.

## Architecture: defense-in-depth for developer workstations

The following is a **proposed** reference architecture, **not** Cursor's deployed design or a tested product integration.

~~~mermaid
flowchart LR
  P["Approved extension registry + provenance policy"] --> E["Managed developer endpoint"]
  E --> X["IDE and isolated extension host"]
  X --> B["Brokered access to Git, CI and cloud"]
  B --> G["Short-lived scoped credentials"]
  E --> T["EDR and device posture telemetry"]
  B --> A["Cloud and Git audit logs"]
  T --> S["Security detection / incident response"]
  A --> S
  S --> Q["Quarantine, revoke, rebuild, verify"]
~~~

See [architecture and design decisions](../../../architecture/2026-10-31-cursor-claude-extension-supply-chain-design-decisions.md) and [standalone context diagram](../../../diagrams/2026-10-31-cursor-claude-extension-context.svg).

### Controls that reduce blast radius

**Provenance:** Maintain a publisher and distribution-channel allowlist. Where supported, pin reviewed extension versions and compare artifact digests to trusted baselines; a signed or familiar listing is not a guarantee that every delivered package is safe.

**Least privilege:** Avoid running the editor with administrator rights. Use per-project, short-lived cloud credentials rather than long-lived organization-admin keys. Keep production deployment credentials out of interactive workstations.

**Isolation:** For unknown repositories or third-party extensions, use disposable dev containers or virtual machines with tightly controlled host mounts, credential forwarding and egress. A dev container is not a complete boundary if it inherits host sockets or secrets.

**Detection:** Correlate extension changes with new SSH authorized keys, LaunchAgents, scheduled tasks, outbound tunnels and unusual cloud/Git access. Prefer central EDR and audit evidence over manual inspection alone.

**Recovery:** Define a rebuild-first policy for confirmed developer endpoint compromise when the host had broad credentials. Track token rotation and session revocation as explicit completion criteria.

## Proposed lab demonstration (no malware required)

Use a **disposable VM** with **dummy credentials only**. Show an empty authorized_keys file, a benign sample line with a harmless marker such as **training-demo-key**, and a read-only detection query against that dummy line. Explain the difference between (a) detecting a file marker, (b) verifying package provenance, and (c) proving compromise. Do **not** install a malicious VSIX, copy an attacker key, run reverse tunnels, or expose real credentials.

A second segment can demonstrate a mock response checklist: device quarantine flag, fake token revocation ticket, and documented reimage approval. Label this as a **simulation**, not a live incident or validated security-product demonstration.

## YouTube production package

- **Primary title:** Claude Code Extension Backdoored in Cursor: Check Your Developer Laptop NOW
- **Alternative:** The Cursor Marketplace Attack: Why Updating Claude Code Isn't Enough
- **Thumbnail:** CURSOR EXTENSION / BACKDOORED? / CHECK NOW
- **Opening hook:** "A developer extension served through Cursor's marketplace tried to leave an SSH backdoor behind. The dangerous part: uninstalling the extension may not remove what it dropped."
- **Target viewers:** Developers, DevOps engineers, platform teams, SOC analysts, IT administrators.
- **Suggested chapters:** Incident facts → affected scope → read-only checks → persistence and token rotation → safer developer architecture.
- **CTA:** "Check the vendor's latest advisory, share the response checklist with your engineering team, and subscribe for source-linked defensive tutorials."
- **Suggested companion assets:** A one-page incident-response checklist and a vendor-agnostic endpoint hardening policy.
- **Timing:** Urgent while remediation guidance is new; update the script if the vendor changes scope or IOCs.

## Limits and editorial checklist

- The exposure estimate is **not** a count of confirmed compromises.
- Cursor's macOS/Windows scope is authoritative for the named packages; some third-party reporting mentions Linux, but this article does not expand the confirmed affected package scope without primary evidence.
- Exact indicators and vendor guidance can change. Re-check before recording and publishing.
- No exploit was reproduced, no endpoint was inspected, and no detection rate is claimed.
- Demand is inferred from an important new incident and coverage; no YouTube search-volume or creator-supply measurement is claimed.

## References (full URLs; primary sources first)

1. Cursor, **Security notice — malicious Anthropic.claude-code versions in Cursor Marketplace**, 29 Oct 2026, updated 30 Oct: https://prod.cursor.com/blog/claude-code-extension-security-notice
2. Reuters, **Cursor says up to 5,000 users affected by tampered Claude Code extension**, 30 Oct 2026: https://www.reuters.com/business/media-telecom/cursor-says-up-5000-users-affected-by-tampered-claude-code-extension-2026-10-30/
3. WIRED, **A Supply Chain Attack Put Malicious Anthropic Claude Extensions on Thousands of Devices**, 30 Oct 2026: https://www.wired.com/story/trojan-malware-targets-thousands-of-anthropic-claude-developer-systems
4. Microsoft, **VS Code Workspace Trust**, accessed 31 Oct 2026: https://code.visualstudio.com/docs/editing/workspaces/workspace-trust
5. Microsoft, **Secure AI-assisted development in VS Code**, accessed 31 Oct 2026: https://code.visualstudio.com/docs/agents/run/security
6. GitHub, **Protecting against security threats**, accessed 31 Oct 2026: https://docs.github.com/en/code-security/tutorials/secure-your-organization/protect-against-threats

*Editorial note: This is defensive educational content, not an incident report from a compromised organization. For affected machines, follow the current vendor advisory and your incident-response procedures.*
