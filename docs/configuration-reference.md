# Configuration Reference

**Complete setup reference for a Mac mini M4 + OpenClaw deployment**

---

## Architecture

```
Mac Mini M4 HOST (macOS Sequoia)
├── Host admin account
├── VM manager (Lume CLI)
│
└── VM: openclaw-vm
    ├── Bridge IP: 192.168.64.x
    ├── Admin account (sudo, setup only)
    ├── Agent account (Standard, no sudo)
    │   ├── OpenClaw workspace: ~/.openclaw/workspace/
    │   └── OpenClaw binary: /opt/homebrew/bin/openclaw
    └── Golden backup snapshot
```

---

## VM Management (run from host)

| Action | Command |
|---|---|
| List VMs | `lume ls` |
| Start VM | `lume run <vm-name>` |
| Stop VM | `lume stop <vm-name>` |
| Clone backup | `lume clone <vm-name> <clone-name>` |
| Delete a clone | `lume delete <clone-name>` |
| SSH as admin | `ssh <admin-user>@192.168.64.x` |
| SSH as agent | `ssh <agent-user>@192.168.64.x` |
| SSH with port forward | `ssh -L 3000:localhost:3000 <agent-user>@192.168.64.x` |

**Important:** `lume run` runs in the foreground. Use `nohup lume run <vm-name> &` for background, or keep the terminal window open.

---

## OpenClaw Configuration (inside VM as agent user)

### Gateway

| Setting | Recommended Value |
|---|---|
| Mode | local |
| Port | 18789 |
| Bind | Loopback (127.0.0.1 only) |
| Auth | Token (via environment variable) |
| Tailscale | Off (keep Tailscale out of the VM) |

Set the gateway token in `~/.zshrc`:
```bash
export OPENCLAW_GATEWAY_TOKEN="your-token-here"
```

**To start gateway:**
```bash
openclaw gateway
```
Leave this terminal open for messaging to work.

**If gateway fails to start:** Must be run from a GUI terminal session inside the VM desktop, not over SSH. VNC in to fix.

### Model Provider

| Setting | Recommended Value |
|---|---|
| Default model | Cheap model (e.g., GPT-5 Nano) for routine tasks |
| Coding model | Premium model (e.g., Claude) for coding tasks |

**To add providers:**
```bash
openclaw models auth paste-token --provider <name>
```

### Heartbeats

| Setting | Recommended Value |
|---|---|
| Interval | Every 120 minutes |
| Active hours | 07:00 - 22:00 |
| Timezone | Your timezone |

**To configure:**
```bash
openclaw config set agents.defaults.heartbeat.every 120m
```

### Workspace Files

| File | Purpose |
|---|---|
| SOUL.md | Agent identity, security boundaries, personality |
| AGENTS.md | Workflow rules, action classification, operating modes |
| TOOLS.md | Tool usage policy, execution mandate |
| USER.md | Operator profile, goals, working style |
| HEARTBEAT.md | Recurring check loop, morning brief, EOD log |
| MEMORY.md | Long-term memory (auto-generated) |

**Deploy workspace files:**
```bash
cat > /tmp/FILENAME.md << 'EOF'
(content)
EOF
scp /tmp/FILENAME.md <agent-user>@192.168.64.x:~/.openclaw/workspace/FILENAME.md
```

### Messaging Channels

**Telegram (recommended):**
1. Create a bot via @BotFather on Telegram
2. Copy the bot token
3. Run: `openclaw configure --section channels`
4. Select Telegram, paste token, set DM policy to "pairing"
5. Restart gateway
6. Message the bot on Telegram — it sends a pairing code
7. Approve: `openclaw pairing approve telegram <code>`

### Tool Permissions

Verify the agent has all necessary tools:
```bash
openclaw agent -m 'List every tool you have access to' --agent main
```

Expected tools: read, write, edit, exec, browser, web_search, web_fetch, cron, message, memory_get, memory_search, subagents.

---

## Quick Reference Commands

| Action | Command |
|---|---|
| Talk to agent | `openclaw agent -m 'message' --agent main` |
| Check status | `openclaw status` |
| Check heartbeat | `openclaw config get agents.defaults.heartbeat` |
| Check gateway | `openclaw config get gateway` |
| View cron jobs | `openclaw cron list` |
| Run diagnostics | `openclaw doctor --fix` |
| Security audit | `openclaw security audit --deep` |
| Update OpenClaw | `openclaw update` |
| Session cleanup | `openclaw sessions cleanup` |

---

## Operational Notes

### Sleep Prevention
The host must not sleep or the VM and gateway stop. Disable sleep:
```bash
sudo pmset -a sleep 0
```

### Gateway Persistence
Currently the gateway must be started manually from a terminal. If the terminal closes, messaging stops. Future improvement: configure as a persistent LaunchAgent.

### Session Context Reset
After significant configuration changes, clear the agent's session to prevent stale context:
```bash
openclaw sessions cleanup --agent main
```

### Golden Backup
Once the system is stable, create a snapshot:
```bash
lume stop <vm-name>
lume clone <vm-name> <vm-name>-GOLDEN-$(date +%Y-%m-%d)
lume run <vm-name>
```

---

## Estimated Monthly Cost

| Category | Model | Cost |
|---|---|---|
| Heartbeats (8/day) | Cheap model | $1-3 |
| Conversations (~20/day) | Cheap model | $2-5 |
| Coding sessions (2-3x/week) | Premium model | $5-15 |
| **Total** | | **$8-23** |

Hardware: ~$600 one-time. No hosting fees.

---

*Built by Alex. March 2026.*
