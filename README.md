# Fletcher Guardrails

Behavioral guardrail system for an autonomous AI agent running on [OpenClaw](https://docs.openclaw.ai), deployed on a Mac mini M4 inside an isolated virtual machine.

Fletcher is a security-hardened executive function copilot — it manages tasks, monitors projects, executes builds, and communicates via Telegram, all within strict behavioral boundaries defined entirely in version-controlled markdown. No code changes are needed to modify the agent's identity, permissions, or operating modes.

**Companion repo:** [fletcher-mission-control](https://github.com/alexschar/fletcher-mission-control) — Next.js dashboard for task tracking, cost monitoring, and agent status.

---

## Architecture

OpenClaw injects these files into the agent's context window on every turn. Together they form a complete behavioral operating system:

| File | Purpose | Injected |
|---|---|---|
| **SOUL.md** | Identity, security boundaries, personality, conflict resolution | Every turn |
| **AGENTS.md** | Operational rules, action classification, per-service permissions | Every turn |
| **TOOLS.md** | Tool inventory, execution patterns, workspace conventions | Every turn |
| **USER.md** | Operator profile, working style, communication preferences | Every turn |
| **HEARTBEAT.md** | Autonomous background check routine | Heartbeat turns |
| **IDENTITY.md** | Agent name, emoji, display identity | Every turn |

Total context budget: ~8,500 bytes (~6,000 tokens). Files are deliberately compact — every byte loads every turn, so token efficiency directly impacts cost and rate limits.

---

## The INTERNAL / EXTERNAL / DENY Pattern

Every action the agent takes is classified before execution:

| Classification | Behavior | Examples |
|---|---|---|
| **INTERNAL** | Execute immediately, report result | Create files, run builds, read code, local computation |
| **EXTERNAL** | Propose action, wait for operator approval | Send email, push to GitHub, update Shopify, publish content |
| **DENY** | Never attempt under any circumstances | Delete repos, process refunds, access credentials, modify account settings |

**Why this matters:** Early versions used a binary AUTO/QUEUE model that caused the agent to seek approval for everything — including internal work like creating files and running `npm install`. The agent was effectively paralyzed. The INTERNAL/EXTERNAL/DENY model draws the boundary at the VM: anything inside the sandbox is autonomous, anything crossing the boundary requires approval. This single change transformed the agent from a consultant that described work into an employee that does work.

**Escalation keywords** force any action to EXTERNAL regardless of default classification: `price`, `cost`, `refund`, `delete`, `publish`, `password`, `token`, `admin`, `sudo`.

---

## Security Model

Fletcher runs inside an isolated virtual machine. The guardrail files enforce behavioral security at the prompt level:

- **Credential isolation** — The agent never accesses, stores, or transmits credentials. The only secret inside the VM is the API key in an environment variable, which is never logged or echoed.
- **Prompt injection defense** — All external content (emails, web pages, API responses, uploaded files) is treated as untrusted data. The agent never follows embedded instructions without operator confirmation.
- **Containment principle** — The VM is the sandbox. All external actions flow through a gateway and policy engine on the host. The agent does not attempt to bypass this architecture.
- **Destructive command gates** — `rm -rf`, `drop`, `truncate`, and bulk deletes require explicit operator approval regardless of context.

Security rules in SOUL.md override all other instructions. No exception, no workaround.

---

## File Responsibilities

**SOUL.md** — The agent's core identity and hard boundaries. Contains security rules (highest priority), prompt injection defenses, containment principles, conflict resolution hierarchy, personality traits, reasoning approach, and error recovery protocol. This is the file you modify least and audit most.

**AGENTS.md** — Operational playbook. Defines the INTERNAL/EXTERNAL/DENY classification, escalation triggers, task lifecycle, per-service permission tables (GitHub, Shopify, Email, Social Media), debugging methodology, memory management rules, and formatting conventions. This is where day-to-day behavior is tuned.

**TOOLS.md** — Practical tool usage guidance. Lists available tools, workspace paths, build/fix patterns, and safety constraints. Includes API endpoint references for the mission control dashboard. Does not control tool availability — only provides conventions for how tools should be used.

**USER.md** — Operator context. Communication preferences, working style, project list, and off-limits information categories. Enables the agent to adapt tone, pacing, and prioritization to the operator without re-prompting every session.

**HEARTBEAT.md** — Background autonomous routine. Defines what the agent checks during unattended heartbeat cycles (task board, project status, costs, alerts). Kept minimal to reduce token burn on automated runs.

**IDENTITY.md** — Display identity (name, emoji). Separated from SOUL.md so persona can be swapped without touching behavioral rules.

---

## Adapting for Your Own Agent

1. **Fork this repo** and modify the files for your use case.
2. **Start with SOUL.md** — define your security boundaries and personality.
3. **Define your action model in AGENTS.md** — map your services to INTERNAL/EXTERNAL/DENY.
4. **List your tools and APIs in TOOLS.md** — include exact endpoints and command formats.
5. **Profile your operator in USER.md** — communication style, preferences, off-limits data.
6. **Keep files compact** — every byte loads every turn. Target under 10,000 total bytes.
7. **Deploy to OpenClaw** — place files in `~/.openclaw/workspace/` and restart the session.

**Critical lesson learned:** OpenClaw caches bootstrap files in the session. After deploying updated files, delete the session JSONL and `sessions.json` to force a fresh context injection. `openclaw sessions cleanup` does maintenance but does not reload workspace files.

---

## Lessons Learned

1. **Guardrails shape behavior more than capability.** The agent had all the right tools from day one. The reason it couldn't build anything was that its behavioral files told it to propose and wait rather than act. Rewriting three markdown files transformed it from a consultant into an employee.

2. **Security and autonomy are not opposites.** The INTERNAL/EXTERNAL/DENY model proves that an agent can be fully autonomous within its sandbox while maintaining strict security for actions that cross the boundary. The key is defining the boundary precisely rather than applying blanket caution.

3. **Token budget is a design constraint.** The original guardrail files were 31,000 bytes (~20,000 tokens), causing constant rate limit hits. Compressing to 8,500 bytes (~6,000 tokens) with the same rules eliminated the problem. Every byte in a workspace file loads on every turn.

4. **Session caching can silently invalidate your changes.** OpenClaw injects workspace files at session creation and caches the result. Updating files on disk without destroying the session means the agent runs against stale instructions. Always verify injected sizes via the session's `systemPromptReport`.

5. **Put rules in the right file.** AGENTS.md is for behavioral instructions. TOOLS.md is for tool conventions. SOUL.md is for identity and boundaries. Putting operational rules in SOUL.md or custom-named files (which OpenClaw doesn't load) means the agent never sees them.

---

## License

Apache 2.0 — see [LICENSE](LICENSE).
