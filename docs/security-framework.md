# Security Framework

**Comprehensive Hardening & Containment Guide for Autonomous AI Agents**

Version 2.0 · March 2026

---

## Executive Summary

This document defines the complete security architecture for running an autonomous AI agent inside a hardened, isolated environment on a Mac mini M4 host. The framework is built on seven core security principles and addresses fifteen identified vulnerability risks.

The guiding philosophy: give the agent exactly what it needs to do its job, and nothing more — then watch everything it does, verify nothing was tampered with, and be ready to kill it instantly.

---

## Architecture Overview

The architecture follows a host-guest isolation model with multiple independent containment layers:

| Layer | Component | Purpose |
|---|---|---|
| Host | Mac mini M4 (macOS) | Operator's primary machine; holds all sensitive data |
| Hypervisor | Apple Virtualization framework | Hardware-level isolation boundary |
| Guest VM | macOS — disposable sandbox | Sandboxed environment for the agent |
| User | Standard account (no sudo) | Non-privileged account with scoped workspace |
| Workspace | ~/workspace/ | Structured directories: input/, output/, logs/ |
| Network | pf firewall + Lulu + DNS filtering | Deny-all with surgical whitelist |
| Monitoring | Host-side log aggregation | Tamper-resistant audit trail |

**Critical: The VM is disposable. If anything looks wrong, snapshot-rollback or full rebuild. Never try to "clean" a potentially compromised VM.**

---

## Principle 1 — Virtual Machine Isolation

The agent never runs directly on the host. A VM creates a hardware-enforced boundary so that even total agent compromise is contained within the VM.

### Configuration

| Component | Setting | Notes |
|---|---|---|
| Hypervisor | Apple Virtualization framework | Native Apple silicon; no emulation overhead |
| Guest OS | macOS — ISOLATED | No iCloud, no Apple ID sign-in |
| User account | Standard (no sudo) | No admin group membership |
| VM Resources | 4 CPU cores, 8GB RAM, 64GB disk cap | Prevents resource exhaustion attacks |

### Snapshot & Rollback Strategy

- **Golden Snapshot:** Create a clean, fully-configured snapshot before any agent session
- **Pre-Session Rollback:** Before each new task session, roll back to the golden snapshot
- **Post-Incident Preservation:** If anomalous behavior is detected, pause the VM, take a forensic snapshot, then rollback to golden

---

## Principle 2 — Network Segmentation

An unrestricted network allows a compromised agent to exfiltrate data, download payloads, or communicate with command-and-control servers.

### Multi-Layer Network Stack

| Layer | Tool | Purpose |
|---|---|---|
| Packet Filter | macOS pf (host) | Kernel-level deny-all; only whitelisted IPs pass |
| App Firewall | Lulu (host) | Alerts on unexpected outbound connections |
| DNS Filter | Custom /etc/hosts or Pi-hole | Blocks DNS tunneling and unauthorized domains |

### Whitelist (Deny-All Default)

| Domain | Purpose | Access Level |
|---|---|---|
| api.anthropic.com | AI backend communication | HTTPS outbound only |
| api.openai.com | AI backend communication | HTTPS outbound only |
| github.com | Code repository access | HTTPS READ-ONLY |

**Critical:** github.com must be restricted to read-only. A compromised agent could push sensitive workspace data to a new public repo. Block git push by removing write-capable SSH keys from the VM and using HTTPS-only clones with no stored credentials.

### DNS Exfiltration Prevention

DNS tunneling is a classic exfiltration channel that bypasses most firewalls. Inside the VM, configure /etc/hosts to only resolve whitelisted domains, and point the VM's DNS resolver to the host. Block all direct DNS to external servers via pf.

---

## Principle 3 — Least Privilege

Over-privileged systems are the root cause of most security breaches. The agent gets the absolute minimum permissions required.

| Component | Setting | Enforcement |
|---|---|---|
| User type | Standard | No sudo, not in admin group |
| File permissions | Workspace only | chmod 700 on workspace root |
| System directories | Read-only at most | SIP protects /System, /usr |
| Package installation | Forbidden | No sudo, no system-level installs |

### Resource Caps

- **CPU:** Never give the agent more than 50% of host cores
- **RAM:** Cap at 8GB; reserve at least 50% for the host
- **Disk:** Set a 64GB disk image cap; alert at 80% full
- **Network:** Rate-limit via pf to detect abnormal transfer volumes

---

## Principle 4 — Authentication & Authorization

### Browser Isolation

| Setting | Value | Rationale |
|---|---|---|
| Profile | Separate isolated profile | No access to operator's sessions |
| Login state | NOT logged into anything | Eliminates session hijacking |
| Mode | Incognito default | No persistent cookies or cache |
| Extensions | None installed | Prevents malicious extension vectors |

### API Key Management

- **Environment variable only:** Never hardcode in scripts or config files
- **Scoped key:** Generate a dedicated API key with usage limits
- **Rotation schedule:** Monthly or immediately after any suspected compromise
- **No key in git:** Pre-commit hooks that scan for leaked secrets

---

## Principle 5 — Monitoring & Auditability

### Disabled Data Channels

| Channel | Status | Rationale |
|---|---|---|
| Shared folders | DISABLED | Prevents host filesystem read/write |
| Clipboard sharing | DISABLED | Prevents cross-boundary data leakage |
| USB passthrough | DISABLED | Prevents access to hardware keys/drives |
| iCloud sync | DISABLED | Prevents cloud-based data exfiltration |

### Host-Side Log Aggregation

Logs stored inside the VM can be tampered with by a compromised agent. The improved architecture streams logs out of the VM in real-time:

- **Syslog forwarding:** VM syslog forwards to the host via UDP on the bridge network
- **Integrity hashing:** Every log batch is SHA-256 hashed on arrival at the host
- **Alert thresholds:** Trigger alerts for: >100 API calls/hour, any outbound connection to non-whitelisted IPs, any sudo attempt, any file access outside workspace

---

## Principle 6 — Kill Switch & Emergency Halt

In an emergency, you need a single command that halts all agent activity instantly.

- **Hard kill:** Force-stop the VM immediately (no graceful shutdown)
- **Network kill:** Flush the firewall whitelist to drop all VM network access without stopping the VM (useful for forensic preservation)
- **Physical kill:** Disconnect Ethernet or disable Wi-Fi as an absolute last resort
- **Keyboard shortcut:** Bind the kill command to a global hotkey for instant access

**Practice the kill procedure before you need it. Run a drill monthly.**

---

## Principle 7 — Human-in-the-Loop Gates

Certain high-risk actions require explicit human approval:

| Action Category | Examples | Approval Method |
|---|---|---|
| Network requests to new domains | Any domain not on whitelist | Prompt to operator |
| File operations outside workspace | Any read/write beyond authorized directory | Blocked by permissions |
| Code execution of downloaded scripts | Running newly-fetched code | Human review required |
| Sending messages to external services | Slack, Discord, Telegram messages | Draft + confirm workflow |
| Any destructive operation | rm -rf, database drops, overwriting files | Explicit confirmation |

As trust builds over time, some gates can be relaxed — but always document which gates were removed and why.

---

## Vulnerability Risks & Mitigations

The framework identifies fifteen specific vulnerability risks:

1. **SSH Brute Force** — Not using VPS. No SSH exposed. Agent runs in local VM only.
2. **Exposed Control Gateway** — All control interfaces local-only. No port forwarding.
3. **Messaging Platform Abuse** — Strict user ID allowlist. Unauthorized messages silently dropped.
4. **Browser Session Hijacking** — Dedicated profile, incognito mode, zero logged-in accounts.
5. **Password Manager Extraction** — No password manager installed in VM.
6. **Integration Token Theft** — Read-only tokens only. All activity monitored. Scoped, not workspace-wide.
7. **Privilege Escalation** — VM + standard user double containment. Guest OS patched.
8. **Prompt Injection** — Input sanitization, size limits, blocklist scanning, behavioral anomaly monitoring, human gate for novel actions.
9. **Malicious Skills/Plugins** — All skills audited and code-reviewed before deployment. Signed manifest.
10. **Data Exfiltration via Git Push** — No SSH keys in VM. HTTPS read-only. Git config locked.
11. **DNS Tunneling Exfiltration** — DNS through host-controlled filter only. Monitor for anomalous queries.
12. **Log Tampering** — Host-side log aggregation with integrity hashing. VM logs supplementary only.
13. **Resource Exhaustion / DoS** — VM resource caps enforced. Host monitoring at 80% thresholds.
14. **API Key Leakage** — Environment variable only. Dedicated scoped key. Pre-commit secret scanning.
15. **Perfect Storm (All Failures)** — Defense in depth. Every layer independent. Kill switch as ultimate backstop.

---

## Forbidden Actions

These are hard restrictions, not guidelines:

| Forbidden Action | Reason | Enforcement |
|---|---|---|
| Connect password managers | Prevents credential exfiltration | Not installed in VM |
| Access host filesystem | Maintains host/guest isolation | No shared folders |
| Run privileged containers | Prevents container escape | Docker not installed; no sudo |
| SSH to production servers | Prevents touching live infrastructure | No SSH keys; port 22 blocked |
| Install system software | Prevents privilege escalation | No sudo; no admin rights |
| Push to git repositories | Prevents data exfiltration | No write credentials |
| Modify its own logging | Prevents audit trail tampering | Logs forwarded to host |
| Access external APIs beyond whitelist | Prevents C2 communication | pf firewall blocks all |

---

## Host-Side Hardening

A compromised host invalidates all VM-level protections.

| Control | Setting | Verification |
|---|---|---|
| Full Disk Encryption | ENABLED | `fdesetup status` |
| System Integrity Protection | ENABLED | `csrutil status` |
| Gatekeeper | ENABLED | `spctl --status` |
| Firewall | ENABLED + stealth mode | System Settings > Network > Firewall |
| Automatic Updates | ENABLED | System Settings > Software Update |
| Screen lock | Require password immediately | System Settings > Lock Screen |

**Run verification commands monthly.**

---

## Operational Procedures

### Daily Startup Checklist
1. Verify host hardening
2. Roll back VM to golden snapshot
3. Verify firewall is active
4. Start the VM, verify standard user is logged in
5. Confirm network whitelist
6. Check log forwarding
7. Begin agent session

### Monthly Maintenance
- Rotate API keys
- Update guest and host OS
- Review and prune network whitelist
- Run a kill switch drill
- Audit all installed skills
- Review this document for updates

---

## Summary

| # | Principle | Key Control |
|---|---|---|
| 1 | VM Isolation | Disposable VM with snapshot/rollback |
| 2 | Network Segmentation | pf + Lulu + DNS; deny-all default |
| 3 | Least Privilege | Standard user, scoped workspace, resource caps |
| 4 | Authentication & Authorization | Isolated browser, strict allowlists, scoped API keys |
| 5 | Monitoring & Auditability | Host-side logs, integrity hashing, disabled data channels |
| 6 | Kill Switch | One-command VM stop, network kill, global hotkey |
| 7 | Human-in-the-Loop Gates | Approval-required for high-risk operations |

No single point of failure. Every layer is independent. Every action is observable. And when something goes wrong, you can kill it in one keystroke.

---

*Built by Alex. March 2026.*
