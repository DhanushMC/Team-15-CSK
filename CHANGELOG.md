# Changelog

All notable changes to **PreMortem + StandupSync** are documented here.
Format: [Keep a Changelog](https://keepachangelog.com/en/1.0.0/)
Versioning: [Semantic Versioning](https://semver.org/spec/v2.0.0.html)

---

## [1.3.0] — 2026-06-16

### Changed (breaking)
- **Split the single combined skill into two independent, composable skills:**
  - `/standup` (`standup-sync`) — standup intelligence: transcript → Claude → Excel → PR checklist
  - `/premortem` (`premortem`) — predictive governance: PR correlation → risk → Teams approval → ArgoCD
- `/premortem` no longer extracts standup tasks. Standup commands (`departments`, `standup`, `checklist`, `sheets sync`) now live under `/standup`.

### Added
- `.claude/commands/standup.md` — `/standup` command entry point
- `.claude/skills/standup-sync/SKILL.md` — canonical StandupSync skill
- `.claude/skills/premortem/SKILL.md` — canonical PreMortem skill
- Dependency graph in `plugin.json`: `premortem` skill `requires` the `standup-intelligence` capability `provides`d by `standup-sync`
- 3-tier Testing Guide in `CLAUDE.md` and `OVERVIEW.md` (instant / demo-mode Actions / full backend)
- Demo-mode guard in `standup-sync.yml` and `pr-analyze.yml` — workflows exit green with a `notice` annotation when `PREMORTEM_BACKEND_URL` is not configured
- `.gitattributes` — excludes `docs/*.html` from GitHub language statistics

### Removed
- `.claude/skills/premortem-standupsync/SKILL.md` — the old combined skill, superseded by the two split skills

### Structure
```
premortem-standupsync v1.3.0
├── .claude/
│   ├── commands/
│   │   ├── standup.md      ← new   (/standup entry)
│   │   └── premortem.md    ← updated (governance only)
│   └── skills/
│       ├── standup-sync/SKILL.md  ← new
│       └── premortem/SKILL.md     ← new
├── docs/                   ← user manuals moved here
└── .github/workflows/      ← demo-mode guards added
```

---

## [1.2.0] — 2026-06-11

### Added
- `plugin.json` — plugin manifest with semantic version, dependency graph, `provides` registry, and SHA-256 file integrity checksums
- `CLAUDE.md` — top-level project context loaded by Claude Code on every session
- `CHANGELOG.md` — this file; version history for skills, hooks, and workflows
- `.claude/skills/premortem-standupsync/SKILL.md` — canonical skill artifact (source of truth for redistribution and child plugins)
- Plugin audit hook in `settings.json` — logs every edit to skill or settings files to `.claude/plugin-audit.log` with timestamp and SHA-256 before/after hash

### Changed
- `.claude/commands/premortem.md` — now clearly separated as the Claude Code command entry layer; canonical architecture lives in `SKILL.md`
- `settings.json` — PostToolUse hook extended to write tamper-evidence entries to `plugin-audit.log` when `SKILL.md` or `settings.json` change

### Structure
```
premortem-standupsync v1.2.0
├── CLAUDE.md           ← new
├── CHANGELOG.md        ← new
├── plugin.json         ← new
├── .claude/
│   ├── settings.json   ← updated
│   ├── commands/premortem.md
│   └── skills/premortem-standupsync/SKILL.md  ← new
└── .github/workflows/  ← unchanged
```

---

## [1.1.0] — 2026-06-11

### Added
- GitHub Actions plugin — 4 enterprise-grade workflows:
  - `standup-checklist.yml` — PR task visibility with pre/post hooks, upsert comment, job summary
  - `pr-analyze.yml` — Claude PR diff ↔ standup task correlation (MATCH / PARTIAL / UNKNOWN), Teams alert on PARTIAL/UNKNOWN
  - `standup-sync.yml` — Scheduled standup transcript sync to Microsoft Excel; manual `workflow_dispatch` with department selector; Teams success/failure cards
  - `remediation-deploy.yml` — AI remediation merge → ArgoCD sync → task Done in Excel → Teams card → audit log; `environment: production` protection gate
- Claude Code hooks in `settings.json`:
  - `SessionStart` — print available `/premortem` commands
  - `PostToolUse: Write|Edit` — log file changes to `skill-activity.log`
  - `PostToolUse: Write` — auto-stage `.py`, `.yml`, `.yaml`, `.md` files
  - `Stop` — print session ready message

---

## [1.0.0] — 2026-05-15

### Added
- `/premortem` Claude Code skill — unified PreMortem + StandupSync architecture
- Multi-department standup intelligence: 10 departments (DevOps, AI Deployment, Testing, INTF, AI Adoption, Finance, Security, Data Governance, SAP, Business)
- Microsoft Excel task registry via MS Graph API (one tab per department, daily upsert)
- PR checklist gate — pending standup tasks injected into GitHub PR body
- PR diff ↔ task correlation (MATCH / PARTIAL / UNKNOWN)
- Infrastructure risk prediction with confidence score
- Microsoft Teams Adaptive Card approval workflow
- AI remediation PR generation (`claude/prevent-<risk>-<task-id>`)
- GitOps deployment via ArgoCD
- User manuals (HTML, print version)

---

## Upgrade notes

### 1.2.0 → 1.3.0
**Breaking — the command surface split in two.** Standup commands moved from `/premortem` to `/standup`:
| Old | New |
|---|---|
| `/premortem departments` | `/standup departments` |
| `/premortem standup [dept] [transcript]` | `/standup standup [dept] [transcript]` |
| `/premortem checklist [dept] [sprint]` | `/standup checklist [dept] [sprint]` |
| `/premortem sheets sync [dept]` | `/standup sheets sync [dept]` |

Governance commands (`risk categories`, `build order`, `explain`, `write prompt for`, `generate checksums`) stay on `/premortem`. After upgrading, run `/premortem generate checksums` to refresh `plugin.json` integrity hashes.

### 1.1.0 → 1.2.0
No breaking changes. Add `plugin.json` and run the checksum hook once to populate integrity hashes:
```
/premortem generate checksums
```

### 1.0.0 → 1.1.0
Add the following GitHub Secrets to your repository before the workflows become active:
- `PREMORTEM_BACKEND_URL`
- `PREMORTEM_API_TOKEN`
- `TEAMS_WEBHOOK_URL`
- `ARGOCD_SERVER_URL` *(remediation-deploy only)*
- `ARGOCD_AUTH_TOKEN` *(remediation-deploy only)*
