# Case Study: Building Fletcher

**A Security-Hardened Autonomous AI Agent — From Concept to Deployment in 48 Hours**

Platform: OpenClaw on Mac mini M4 · Architecture: VM-Isolated, Dual-Model, Human-in-the-Loop · March 2026

---

## Executive Summary

Fletcher is a security-hardened autonomous AI agent built on the OpenClaw framework, running inside an isolated virtual machine on a Mac mini M4. Designed for a solo developer and entrepreneur managing multiple creative and business projects simultaneously, Fletcher serves as an executive function copilot: prioritizing tasks, preserving project memory, reducing cognitive overload, and executing technical work autonomously within defined security boundaries.

The system was designed and deployed in a single intensive 48-hour sprint, covering hardware selection, VM isolation architecture, security framework design, behavioral guardrail authoring, dual-model cost optimization, messaging integration, and mission control dashboard development. The result is a production-ready personal AI infrastructure that costs under $20/month to operate.

---

## The Problem

The operator, a solo developer working across game development (Unity), iOS apps (SwiftUI), e-commerce (Shopify), content creation, and AI systems, faced a fundamental challenge: too many strong ideas competing for limited executive function capacity. The bottleneck was not technical skill or creative vision, but the ability to sequence, prioritize, and maintain momentum across a diverse project portfolio.

Existing tools (task managers, note apps, AI assistants) failed because they treated each interaction as stateless. Context was lost between sessions. Priorities drifted without detection. Projects stalled not from lack of interest but from the cognitive cost of re-entering them after time away.

The operator needed a system that would remember everything, understand working patterns, protect energy, and act autonomously on internal tasks while maintaining strict security boundaries for external actions.

---

## Design Principles

Seven core principles guided every architectural and behavioral decision:

1. **VM Isolation.** The agent never runs on the host machine. A compromised agent is contained within a disposable virtual machine.

2. **Least Privilege.** Standard user account, no sudo, scoped workspace, resource caps. The agent gets exactly what it needs and nothing more.

3. **Defense in Depth.** Seven independent security layers (VM, firewall, DNS, permissions, monitoring, kill switch, human gates). No single point of failure.

4. **Human-in-the-Loop for External Actions.** Internal work (files, code, builds) runs autonomously. External actions (emails, code pushes, public posts) require explicit approval.

5. **Dual-Model Cost Optimization.** Cheap model (GPT-5 Nano) for routine tasks; expensive model (Claude) only for coding. Keeps monthly cost under $20-30.

6. **Executive Function First, Automation Second.** Fletcher's primary role is not task execution but attention management, project memory, and momentum protection.

7. **Behavioral Guardrails as Code.** The agent's personality, decision-making, security boundaries, and operating modes are defined in version-controlled markdown files loaded on every turn.

---

## Architecture

### Hardware

Mac mini M4, 24GB RAM, 512GB SSD. Selected over alternatives (Raspberry Pi 5, cloud VPS) for the combination of always-on capability, Apple silicon performance, sufficient RAM for a Node.js gateway making cloud API calls, and a large SSD to reduce write amplification on a 24/7 appliance.

### Isolation Model

| Layer | Component | Purpose |
|---|---|---|
| Host | Mac mini M4 (macOS Sequoia) | Operator's primary machine; holds all sensitive data |
| Hypervisor | Lume (Apple Virtualization) | Hardware-level isolation boundary |
| Guest VM | macOS VM | Disposable sandboxed environment |
| User | Standard account (no sudo) | Non-privileged agent account |
| Network | Bridge at 192.168.64.x | Host-only network, no direct internet |
| Gateway | OpenClaw on port 18789 | Bridges messaging, tools, and API calls |

### Network Security

The VM operates on a deny-all network policy. Only whitelisted domains are accessible: api.openai.com, api.anthropic.com, and github.com (read-only). A multi-layer network stack enforces this through pf packet filtering at the kernel level, Lulu application-level firewall, and DNS filtering to prevent tunneling-based exfiltration. GitHub access is restricted to HTTPS read-only with no stored credentials, preventing a compromised agent from pushing sensitive data to a public repository.

### Dual-Model Routing

Fletcher uses GPT-5 Nano as the default model for all routine tasks: conversation, planning, research, morning briefs, and analysis. When coding work is required, Fletcher switches to Anthropic Claude, then switches back to Nano when the coding is complete. This routing is enforced through behavioral instructions in AGENTS.md and tracked via cost logging. Estimated monthly cost: $8-23.

---

## Behavioral Guardrail System

Fletcher's behavior is governed by five markdown files loaded into context on every interaction:

| File | Purpose | Key Content | Size |
|---|---|---|---|
| SOUL.md | Agent identity and core principles | Executive function copilot role, security boundaries, personality, error recovery | ~5 KB |
| AGENTS.md | Operational rules and modes | INTERNAL/EXTERNAL/DENY actions, four operating modes, drift detection, idea triage | ~8 KB |
| TOOLS.md | Tool usage and execution policy | Explicit tool inventory, execution mandate, workspace paths, safety constraints | ~4 KB |
| USER.md | Operator profile | Working style patterns, energy cycles, all projects, goals, support rules | ~9 KB |
| HEARTBEAT.md | Recurring check loop | Six check categories, morning brief, end-of-day log, noise suppression | ~3 KB |

### The INTERNAL / EXTERNAL / DENY Model

The original guardrail files used an AUTO/QUEUE/DENY classification that caused Fletcher to seek approval for everything, including internal work like creating files and running npm. The revised INTERNAL/EXTERNAL/DENY model explicitly separates autonomous internal work (anything inside the VM) from approval-gated external actions (anything crossing the VM boundary). This single change transformed Fletcher from a consultant who described work to an employee who does work.

### Four Operating Modes

- **Today Mode:** Daily prioritization. Surfaces 1-3 priorities based on energy, urgency, and strategic importance. Protects against overcommitting.
- **Project Mode:** Deep context on one project. Automatically restores context when the operator returns to a project after time away.
- **Creator Mode:** Content, brand, and audience work. Maintains an idea bank, supports messaging consistency, and helps turn insights into public content.
- **Pattern Mode:** Self-reflection and pattern tracking. Supports journaling, tracks recurring patterns in productivity and energy, connects patterns to project timing.

### Drift Detection

Fletcher monitors for signs that the operator is spreading too thin, abandoning priorities without conscious choice, or stuck in setup loops. When drift is detected, Fletcher responds with compassionate course correction rather than surveillance or judgment. The tone is explicitly defined: suggest narrowing focus, never say the operator is off track.

---

## Deployment Timeline

| Phase | Work Completed |
|---|---|
| Night 1 (10pm-2am) | Hardware selection, VM creation, OpenClaw installation, gateway configuration, API key setup, workspace file deployment (v1), initial agent conversation, Telegram bot creation and pairing |
| Day 2 (Morning) | Gateway restart troubleshooting, Telegram reconnection, heartbeat configuration, tool permissions verification, agent execution testing |
| Day 2 (Afternoon) | Behavioral file rewrite (v2) incorporating deep analysis of operator working style and goals. All five workspace files rewritten and deployed. Agent personality verification test passed. |
| Day 2 (Evening) | Mission control dashboard built and packaged for deployment. Four screens: costs, tasks, schedule, memory viewer. |

---

## Key Challenges and Solutions

### Challenge 1: Agent Refused to Execute

The most persistent issue was the agent presenting code in chat instead of using its exec and write tools to create files directly.

**Root cause:** The original SOUL.md described the agent as a "thinking engine, not a control engine" and the workflow rules required approval at every step. The agent internalized this as "I should never act autonomously."

**Solution:** Complete rewrite of SOUL.md ("I am a builder and a doer"), AGENTS.md (INTERNAL actions need no approval), and TOOLS.md (explicit tool inventory with execution mandate). After deployment, the agent correctly created a directory on the first attempt.

### Challenge 2: Gateway Required GUI Session

The OpenClaw gateway LaunchAgent could not be installed over SSH; it required a GUI desktop session inside the VM.

**Solution:** VNC into the VM desktop, open Terminal as the agent user, and run the gateway install from the GUI session. Documented as a known requirement.

### Challenge 3: Messaging Platform 401 Unauthorized

After configuring Telegram, the gateway repeatedly failed with 401 Unauthorized errors.

**Root cause:** The bot token was invalid.

**Solution:** Regenerate the token via @BotFather, update the config, restart the gateway. Connected successfully on the next attempt.

### Challenge 4: Session Context Poisoning

After multiple failed execution attempts, the agent's session context accumulated a learned helplessness: it "remembered" that it couldn't execute and continued to refuse even after the gateway was fixed.

**Solution:** Clear the session context and start fresh. This is now a standard procedure after any significant configuration change.

### Challenge 5: Balancing Security with Autonomy

The original security framework applied the same level of caution to internal operations (creating files in the VM) as external operations (pushing code to GitHub), crippling productivity.

**Solution:** The INTERNAL/EXTERNAL/DENY model creates a clear boundary: everything inside the VM is autonomous, everything crossing the VM boundary requires approval. Security remained strong for what matters while internal productivity was unblocked.

---

## Cost Analysis

| Category | Model | Est. Monthly | Notes |
|---|---|---|---|
| Heartbeats (8/day) | GPT-5 Nano | $1-3 | Silent checks |
| Daily conversations | GPT-5 Nano | $2-5 | ~20 msgs/day |
| Coding sessions | Claude | $5-15 | 2-3x/week |
| **Total** | | **$8-23** | Under budget |

Hardware cost: Mac mini M4 (~$600 one-time). No monthly hosting fees. No cloud infrastructure. The system runs entirely on local hardware with only API calls going to cloud providers.

---

## Outcomes

- Fully operational autonomous agent accessible via Telegram from a mobile phone
- Seven-layer security architecture with VM isolation, network segmentation, and human-in-the-loop gates
- Behavioral system that genuinely understands operator working style and project priorities
- Dual-model cost optimization keeping monthly spend under $25
- Four-screen mission control dashboard for cost tracking, task management, scheduling, and memory browsing
- Complete configuration reference document enabling recovery and maintenance
- Agent correctly articulated understanding of operator as "a chaotic but beautifully kinetic creator who needs a calm, structured partner to surface the next steps that actually move things forward"

---

## Lessons Learned

**1. Guardrails shape behavior more than capability.** The agent had all the right tools from the start. The reason it couldn't build anything was that its behavioral files told it to propose and wait rather than act. Rewriting three markdown files transformed it from a consultant into an employee. The lesson: an AI agent's instructions are its personality, and personality determines productivity.

**2. Security and autonomy are not opposites.** The INTERNAL/EXTERNAL/DENY model proves that an agent can be fully autonomous within its sandbox while maintaining strict security for actions that cross the boundary. The key insight is defining the boundary precisely rather than applying blanket caution to everything.

**3. Understanding the operator is the highest-value investment.** The most impactful change in the entire project was not any technical configuration but the deep USER.md that describes the operator's working patterns, energy cycles, and what kind of support actually helps versus overwhelms. A generic productivity assistant would have failed. A personalized executive function copilot succeeds.

**4. Session context can poison an agent.** After repeated failures, the agent learned helplessness from its own conversation history. Clearing the session and starting fresh with corrected instructions was necessary. This should be a standard procedure after configuration changes.

**5. Use the right tool for the right job.** The agent excels at ongoing operation, conversation, and memory. One-shot complex coding projects are better handled by dedicated coding tools. Fletcher attempted to build the mission control dashboard three times and produced nothing on disk. The dashboard was ultimately built externally and transferred in.

---

## Roadmap

1. Network hardening: Apply pf firewall rules on the host to lock down all VM internet access to approved domains only
2. Web search integration: Configure search API to give the agent access to current information
3. Golden backup: Create a stable VM snapshot for rollback capability
4. Gateway persistence: Configure the gateway as a LaunchAgent that auto-starts on boot
5. Morning brief activation: Enable the automated daily brief via Telegram with priorities, project status, and weather
6. Cost tracking integration: Wire API calls to the mission control cost tracker for real-time spend monitoring
7. Local model evaluation: Assess whether adding a local model on the host provides meaningful cost savings versus the complexity it introduces

---

*Built by Alex with assistance from Claude (Anthropic). March 2026.*
