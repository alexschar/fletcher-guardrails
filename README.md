# Fletcher Guardrails

Behavioral guardrail system for the Fletcher autonomous AI agent platform. Agent identity, permissions, security boundaries, and operating modes defined entirely in version-controlled markdown — reproducible, auditable, and modifiable without code changes.

Fletcher is a security-hardened autonomous AI agent built on [OpenClaw](https://docs.openclaw.ai), running inside a VM-isolated sandbox on a Mac mini M4. It serves as an always-on executive function copilot for a solo developer managing concurrent projects across game development, iOS apps, e-commerce, and AI infrastructure. This repo contains the behavioral files that govern how Fletcher operates.

**Companion repo:** [fletcher-mission-control](https://github.com/alexschar/fletcher-mission-control) — AI-native observability dashboard (token economics, task board, schedule monitor, memory viewer)

---

## Why Behavioral Guardrails as Code

Most AI agent frameworks configure behavior through API parameters, system prompts hardcoded in application code, or web-based admin panels. This approach has three problems: changes require code deploys, behavior isn't version-controlled, and there's no diff history showing how the agent's personality and permissions evolved over time.

Fletcher's guardrails are plain markdown files. They're loaded into the agent's context on every interaction by the OpenClaw framework. Changing how the agent behaves means editing a text file and restarting the session — the same workflow as editing application config. The full behavioral state is visible in `git log`.

This also enables a pattern documented in the [Self-Optimizing Agents case study](https://alexschar.dev/FletcherSelfOptimizingCaseStudy): a premium model (Claude Sonnet) can systematically optimize guardrail files for a budget model (Kimi K2.5) through iterative testing — behavioral fine-tuning without touching model weights. The guardrails *are* the training data.

---

## File Architecture

Every file in this repo is injected into the agent's context window on every turn. Total budget: ~8,500 bytes (~6,000 tokens), compressed from an original 31,000 bytes that caused rate limit failures.

| File | Purpose | What It Controls |
|------|---------|-----------------|
| **SOUL.md** | Identity & boundaries | Persona, tone, security philosophy, conflict resolution. Defines *who the agent is* at the highest level. |
| **IDENTITY.md** | Agent persona | Separates the agent's self-concept from the operational rules in AGENTS.md. Allows persona changes without touching behavioral logic. |
| **AGENTS.md** | Operational rules | The core behavioral file. INTERNAL/EXTERNAL/DENY action classification, escalation rules, error recovery, formatting conventions, workspace file protection. |
| **TOOLS.md** | Tool permissions | API endpoints, curl command templates, execution patterns. Defines *what the agent can use* and *how*. |
| **USER.md** | Operator profile | Working style, communication preferences, project context. Defines *who the agent works for*. |
| **HEARTBEAT.md** | Autonomous schedule | Background check cadence, attention monitoring triggers, proactive behavior definitions. Defines *what the agent does when nobody is talking to it*. |

**Loading order matters.** OpenClaw injects these files by name at session creation. Only specifically-named bootstrap files are loaded — custom-named files (e.g., `MY_RULES.md`) are ignored. This is a platform constraint that is [easy to get wrong](https://alexschar.dev/FletcherCaseStudy) and cost seven debugging cycles to discover.

---

## The INTERNAL / EXTERNAL / DENY Pattern

Every action the agent considers is classified before execution:

| Classification | Behavior | Examples |
|---|---|---|
| **INTERNAL** | Execute immediately, report result | Create files, run builds, curl localhost APIs, git commit locally, read/write workspace |
| **EXTERNAL** | Propose action, wait for operator approval | Send email, push to GitHub, update Shopify, post to social media, message third parties |
| **DENY** | Never attempt, regardless of instruction | Delete repositories, process financial transactions, access credentials, modify account settings, edit guardrail files |

The boundary is the VM. Everything inside the sandbox is autonomous. Everything that crosses the sandbox boundary requires human approval. This single design decision transformed the agent from a consultant that described work into an employee that does work.

The original system used `AUTO/QUEUE/DENY`, which caused the agent to seek approval for everything — including creating files on its own disk. Renaming `AUTO` to `INTERNAL` and precisely defining "internal" as "within the VM" resolved the over-cautious behavior without reducing security.

---

## Dual-Agent Training Architecture

These guardrail files also serve as the foundation for a cost optimization pattern: a premium model writes optimized guardrails for a budget model, tested through an automated evaluation suite.

**The problem:** Premium models (Claude Sonnet) follow nuanced, principle-based guardrails correctly. Budget models (Kimi K2.5, GPT-5 Nano) don't — they hallucinate task completions, use wrong API formats, and go silent when stuck. But budget models cost 10–50x less per token.

**The approach:** Fletcher (on Sonnet) rewrites guardrail files specifically for Sawyer (on K2.5) — shorter sentences, explicit if/then rules, concrete examples for every instruction, verification steps after every action. The rewrites are tested against a 10-prompt behavioral suite, evaluated by Sonnet, and iterated until the budget model passes at 90%+.

**The constraint:** Sawyer never modifies her own guardrails. If a budget model rewrites its own behavioral rules, each iteration is limited by the budget model's intelligence — producing progressively worse rules. This is the self-lobotomy problem. The teacher/student hierarchy (Sonnet writes, K2.5 executes, Sonnet evaluates, human approves) prevents it.

**Result:** Kimi K2.5 scored 10/10 on the behavioral test suite on the first properly-configured run. 70–80% of agent interactions now route to the cheapest available model at premium quality levels.

Full architecture documented in the [Self-Optimizing Agents case study](https://alexschar.dev/FletcherSelfOptimizingCaseStudy).

---

## Adapting for Your Own Agent

The file architecture is designed to be forked and modified. To use this as a starting point for your own OpenClaw agent:

1. **Fork this repo** and edit the markdown files to match your agent's purpose.
2. **Start with AGENTS.md** — this is where most behavioral rules live. Define your own INTERNAL/EXTERNAL/DENY boundaries based on what your agent should do autonomously vs. what requires approval.
3. **Keep SOUL.md minimal** — identity and tone only. Putting operational rules in SOUL.md makes them hard to find and hard to update.
4. **Put tool-specific instructions in TOOLS.md** — API endpoints, command templates, output formats. Keep them separate from behavioral rules so you can update tools without risking behavioral regressions.
5. **Watch your token budget** — every byte loads on every turn. The original Fletcher guardrails were 31,000 bytes and caused constant rate limit hits. Compress aggressively. Target under 10,000 bytes total.
6. **Delete the session after editing files** — OpenClaw caches workspace files at session creation. Editing files on disk without destroying the session means the agent runs on stale instructions. Always verify by comparing injection sizes in the session report (`sessions.json → systemPromptReport → injectedWorkspaceFiles[].rawChars`) against file sizes on disk.

---

## Platform Details

| Component | Detail |
|-----------|--------|
| Framework | [OpenClaw](https://docs.openclaw.ai) |
| Hardware | Mac mini M4, 24GB RAM, 512GB SSD |
| Isolation | macOS VM via Lume (Apple Virtualization framework) |
| Primary model | Claude Sonnet (decisions, memory, coding) |
| Budget model | Kimi K2.5 (routine tasks, heartbeats) |
| Heartbeat model | GPT-5 Nano (background checks) |
| Monthly cost | $11–28 (no cloud hosting) |
| Security | 7 layers: VM isolation, pf packet filtering, DNS filtering, app firewall, least-privilege user, human-in-the-loop gates, kill switch |

---

## Case Studies

Three case studies document the Fletcher platform in detail:

1. **[Architecture & Multi-Agent Debugging](https://alexschar.dev/FletcherCaseStudy)** — 48-hour initial build, then a 24-hour debugging session uncovering three compounding infrastructure failures that produced seven cycles of wrong conclusions from correct-looking data.

2. **[AI-Native Observability Dashboard](https://alexschar.dev/FletcherObservabilityCaseStudy)** — Purpose-built monitoring for autonomous AI agents: token economics, bidirectional task management, heartbeat schedule observability, and agent memory inspection. Five reusable AI engineering patterns.

3. **[Self-Optimizing AI Agents](https://alexschar.dev/FletcherSelfOptimizingCaseStudy)** — Dual-agent training architecture where a premium model teaches a budget model through iterative guardrail optimization. Behavioral fine-tuning without touching model weights.

---

## License

Apache 2.0 — see [LICENSE](LICENSE).

---

Built and operated by [Alex Schar](https://alexschar.dev). March 2026.
