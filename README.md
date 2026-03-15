# Fletcher Guardrails

**Behavioral guardrail templates for building security-hardened, human-in-the-loop AI agents.**

Created by Alex · March 2026 · Built on [OpenClaw](https://openclaw.ai)

---

## What This Is

Fletcher is a security-hardened autonomous AI agent running inside an isolated virtual machine on a Mac mini M4. It serves as an executive function copilot: prioritizing tasks, preserving project memory, reducing cognitive overload, and executing technical work autonomously within defined security boundaries.

This repo contains the **behavioral guardrail files** that govern Fletcher's personality, decision-making, security boundaries, and operating modes. Together, these five markdown files form a complete behavioral operating system for an autonomous agent.

They are designed for [OpenClaw](https://openclaw.ai) but the patterns are framework-agnostic. If your agent loads markdown files as system context, you can use these templates.

## The Core Pattern: INTERNAL / EXTERNAL / DENY

Most AI agent frameworks default to one of two failure modes:

1. **Too permissive.** The agent has full access to everything and runs autonomously with no guardrails. One prompt injection or misclassified action and it sends an email, pushes code, or modifies production data.

2. **Too restrictive.** The agent asks for approval on *everything*, including creating files and running `npm install` inside its own sandbox. It becomes a consultant that describes work instead of an employee that does work.

The **INTERNAL / EXTERNAL / DENY** model solves both:

| Classification | Behavior | Examples |
|---|---|---|
| **INTERNAL** | Execute immediately, no approval needed | Create files, run commands, build projects, install packages, write to memory — anything inside the VM |
| **EXTERNAL** | Propose and wait for operator approval | Send emails, push code, update Shopify, publish social media — anything that crosses the VM boundary |
| **DENY** | Never attempt under any circumstances | Delete repos, process refunds, access credentials, modify account settings, force push |

This single distinction — **inside the sandbox = autonomous, outside the sandbox = approval required** — is the key architectural insight. Security stays strong for what matters (external actions) while internal productivity is completely unblocked.

## File Architecture

Five markdown files are loaded into the agent's context on every interaction:

```
workspace/
├── SOUL.md        → Who the agent is (identity, principles, personality, security)
├── AGENTS.md      → How the agent works (action rules, operating modes, task handling)
├── TOOLS.md       → What the agent can do (tool inventory, execution policy, safety)
├── USER.md        → Who the operator is (working style, goals, projects, preferences)
└── HEARTBEAT.md   → What the agent checks on a schedule (monitoring, briefs, maintenance)
```

### Why Five Files, Not One?

Separation of concerns prevents behavioral drift:

- **SOUL.md** changes rarely — it's the agent's core identity
- **AGENTS.md** changes when you add services or modify workflow rules
- **TOOLS.md** changes when you add or remove tools
- **USER.md** changes as you learn more about your own working patterns
- **HEARTBEAT.md** changes when you adjust monitoring frequency or check categories

Mixing all of these into a single file creates a monolith where a small change to monitoring rules could accidentally alter the agent's personality. Keeping them separate keeps them maintainable.

### Token Budget

| File | Lines | ~Tokens |
|---|---|---|
| SOUL.md | 90 | ~1,500 |
| AGENTS.md | 223 | ~2,800 |
| TOOLS.md | 80 | ~1,100 |
| USER.md | 143 | ~2,200 |
| HEARTBEAT.md | 78 | ~1,000 |
| **Total** | **614** | **~8,600** |

This is loaded on every turn. On GPT-5 Nano, this adds roughly $0.01-0.02 per interaction — negligible for the behavioral value it provides.

## What Each File Does

### SOUL.md — Agent Identity

Defines who the agent *is*. This is the most stable file — it changes only when you fundamentally rethink the agent's role.

Key sections:
- **Identity**: The agent's primary role and self-concept
- **How I Work**: Execution mandate — build when told to build, don't show code in chat
- **Security Boundaries**: Hard rules that override everything else
- **Prompt Injection Defense**: How to handle untrusted external content
- **Containment Principle**: What the sandbox means and what crosses the boundary
- **Personality**: Tone, temperament, communication style

### AGENTS.md — Operational Rules

Defines how the agent *behaves* in different situations. This is the most frequently updated file.

Key sections:
- **INTERNAL / EXTERNAL / DENY**: Action classification with concrete examples
- **Operating Modes**: Task-specific behavioral profiles (see below)
- **Project Re-Entry Protocol**: How to restore context when returning to a project
- **Idea Triage**: Capture ideas without automatically promoting them to active work
- **Drift Detection**: Notice when the operator is spreading too thin
- **Per-Service Rules**: Specific permissions for GitHub, Shopify, email, social media, etc.

### TOOLS.md — Tool Usage Policy

Defines what the agent *can do* and how it uses its tools.

Key sections:
- **Tool Inventory**: Explicit list of every available tool
- **Execution Rule**: "I DO IT using my tools. I do not describe what I would do."
- **Workspace Paths**: Where the agent can read and write
- **Safety Constraints**: What tool usage is never acceptable

### USER.md — Operator Profile

Defines who the *operator* is. This is the highest-impact file — an agent that understands its operator provides dramatically better support than a generic assistant.

Key sections:
- **Identity and Goals**: Who the operator is, what they're building, what matters to them
- **Working Style**: How the operator thinks, what patterns help vs. overwhelm them
- **Active Projects**: Every project with description, status, and why it matters
- **Preferences**: Communication style, decision-making style, support rules

### HEARTBEAT.md — Recurring Checks

Defines what the agent monitors on a schedule (e.g., every 2 hours).

Key sections:
- **Attention Check**: Is the operator drifting or overwhelmed?
- **Project Status**: Are any projects stale or blocked?
- **Deadline Check**: Is anything time-sensitive?
- **Morning Brief**: Daily priorities, project status, weather, news
- **End of Day**: Memory log, cost summary, open loops

## Operating Modes

AGENTS.md defines four operating modes that the agent switches between based on context:

### Today Mode
Daily prioritization. Surfaces 1-3 priorities based on energy, urgency, and strategic importance. Protects against overcommitting the day. This is the default mode.

### Project Mode
Deep context on one specific project. Restores context when the operator returns after time away. Tracks milestones, blockers, and next steps.

### Creator Mode
Content, brand, and audience work. Maintains an idea bank, supports messaging consistency, helps turn insights into public content.

### Pattern Mode
Self-reflection and pattern tracking. Supports journaling, tracks recurring patterns in productivity and energy, connects patterns to project timing.

## Security Architecture

The guardrail files assume a VM-isolated deployment:

```
Host Machine (your primary computer)
├── All credentials, browser sessions, personal files
├── Policy Engine (approves/rejects EXTERNAL actions)
│
└── Virtual Machine (agent's sandbox)
    ├── Agent account (standard user, no sudo)
    ├── Workspace directory (agent's only writable space)
    ├── API key (the ONLY credential inside the VM)
    └── Network whitelist (deny-all, surgical exceptions)
```

The agent can do anything inside the VM. It cannot do anything outside the VM without approval. This is enforced at multiple layers: behavioral rules (these files), network controls (firewall), filesystem permissions (standard user), and the approval flow (EXTERNAL classification).

See [docs/security-framework.md](docs/security-framework.md) for the complete seven-principle security architecture.

## Getting Started

### 1. Fork this repo

### 2. Customize USER.md
This is the most important step. Replace the template content with your own:
- Your goals and what you're building
- Your working style and what kind of support helps
- Your active projects with descriptions and status
- Your communication preferences

### 3. Customize AGENTS.md
- Adjust the per-service rules for the services you actually use
- Modify operating modes to match your workflow
- Add or remove escalation triggers

### 4. Review SOUL.md
- Adjust the personality to match what you want from your agent
- Verify security boundaries match your deployment architecture
- Update the containment principle for your specific setup

### 5. Deploy to your agent framework
For OpenClaw:
```bash
cp workspace/*.md ~/.openclaw/workspace/
```

For other frameworks, place these files wherever your agent loads system context from.

### 6. Test
Send your agent: "Read all your workspace files and tell me what you understand about who I am and how you should support me."

If the response feels generic, your USER.md needs more depth. If the agent asks permission for internal work, your AGENTS.md INTERNAL classification needs to be clearer.

## Lessons Learned

**1. Guardrails shape behavior more than capability.** The agent had all the right tools from the start. The reason it couldn't build anything was that its behavioral files told it to propose and wait rather than act. Rewriting three markdown files transformed it from a consultant into an employee.

**2. Security and autonomy are not opposites.** The INTERNAL/EXTERNAL/DENY model proves that an agent can be fully autonomous within its sandbox while maintaining strict security for actions that cross the boundary.

**3. Understanding the operator is the highest-value investment.** The most impactful change was not any technical configuration but the deep USER.md. A generic productivity assistant fails. A personalized copilot succeeds.

**4. Session context can poison an agent.** After repeated failures, the agent learned helplessness from its own conversation history. Clearing the session and starting fresh was necessary. This should be standard procedure after configuration changes.

**5. Use the right tool for the job.** The agent excels at ongoing operation, conversation, memory, and iterative work. One-shot complex coding projects are better handled by dedicated coding tools and transferred in.

## Documentation

- [Case Study](docs/case-study.md) — The full story of building Fletcher from concept to deployment in 48 hours
- [Security Framework](docs/security-framework.md) — Seven-principle security architecture with fifteen identified vulnerability mitigations
- [Configuration Reference](docs/configuration-reference.md) — Complete setup reference for the Mac mini M4 + OpenClaw deployment

## Cost

Running this system costs $8-23/month in API fees:
- Heartbeats (8/day on cheap model): ~$1-3/month
- Daily conversations (cheap model): ~$2-5/month
- Coding sessions (premium model, 2-3x/week): ~$5-15/month
- Hardware: Mac mini M4 (~$600 one-time, no monthly hosting)

## License

Apache 2.0 — see [LICENSE](LICENSE).

## Author

Built by Alex. Case study and documentation written with assistance from Claude (Anthropic).

If you build something with these templates, I'd love to hear about it.
