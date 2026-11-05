# PATA — a Claude Code skill

**PATA** is an app development framework by **Francis Chin** ([World Impact](https://github.com/pacochin88)). Every table, field and piece of business logic in an app belongs to one of four layers:

| Layer | What lives here | Examples |
|---|---|---|
| **P — Parameters** | Choices the business controls without a deploy | categories, statuses, roles, currencies |
| **A — Assets** | The core "things" the app manages | users, products, listings, profiles |
| **T — Transactions** | Records of something happening | bookings, orders, payments, reviews |
| **A — Analytics** | Insights derived from the other three | revenue by category, trends, dashboards |

**The golden rule: nothing configurable is hard-coded.** If it shapes choices, filtering, categorisation or display, it lives in a Parameters table that an admin can manage. It never goes in the code.

Read the full framework and the reasoning behind it in [`references/pata-framework.md`](references/pata-framework.md).

## What the skill does

Once installed, Claude Code applies PATA automatically:

- **New project:** drafts a PATA Map (every table sorted into P / A / T / A) for your approval, then builds Parameters first, with starter data and admin screens.
- **New feature:** sorts every new table and field into its layer before writing any code.
- **Existing project audit:** finds hard-coded lists, enums and magic strings, then proposes a fix plan before changing anything.
- Keeps a living `docs/PATA.md` blueprint in every project.

## Install

Install it once and it works in every project.

**Mac / Linux**
```bash
git clone https://github.com/pacochin88/pata-skill.git ~/.claude/skills/pata
```

**Windows (PowerShell)**
```powershell
git clone https://github.com/pacochin88/pata-skill.git "$env:USERPROFILE\.claude\skills\pata"
```

Restart Claude Code after installing.

**Update to the latest version:**
```bash
cd ~/.claude/skills/pata && git pull
```

## How to use it

- **Call it directly:** type `/pata` in Claude Code.
- **Or just ask:** for example, *"Start this app with PATA"*, *"Add a booking feature"* or *"Check this project for hard-coded values"*.

**Always on (optional):** add this line to `~/.claude/CLAUDE.md`:
```
Always apply the PATA skill when designing databases, adding features, or starting new projects.
```

## Files

```
SKILL.md                     ← instructions Claude Code follows
references/pata-framework.md ← the full PATA framework
```
