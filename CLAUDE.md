# PreMortem + StandupSync — Plugin Context

## What this plugin does

Transforms DEWA's daily standup meetings into a fully automated, AI-governed DevOps and business operations platform.

```
Daily Standup (Teams) → Transcript → Claude → Tasks → Excel
        ↓
Tasks become PR Checklists → GitHub PR Gate → Risk Analysis → Remediation → Human Approval → GitOps Deploy
```

## Plugin structure

```
premortem-standupsync/
├── CLAUDE.md                               ← you are here
├── CHANGELOG.md                            ← version history
├── plugin.json                             ← manifest: version, deps, checksums
├── .claude/
│   ├── settings.json                       ← Claude Code hooks
│   ├── commands/
│   │   └── premortem.md                    ← /premortem slash command entry
│   └── skills/
│       └── premortem-standupsync/
│           └── SKILL.md                    ← canonical skill (full architecture)
└── .github/
    └── workflows/
        ├── standup-checklist.yml           ← PR task visibility
        ├── pr-analyze.yml                  ← Claude PR ↔ task correlation
        ├── standup-sync.yml                ← scheduled standup → Excel
        └── remediation-deploy.yml          ← AI fix → ArgoCD gate
```

## Core safety principle

> **AI Suggests → Human Approves → GitOps Deploys**
> AI never deploys directly. Every remediation requires explicit human approval via Teams Adaptive Card.

## Skill entry point

Run `/premortem` in Claude Code. Supported commands:

| Command | What it does |
|---|---|
| `/premortem departments` | List all 10 departments and their task schemas |
| `/premortem build order` | Return the 4-phase build sequence |
| `/premortem generate [module]` | Generate complete Python code for a named module |
| `/premortem standup [dept] [transcript]` | Extract tasks from a transcript |
| `/premortem checklist [dept] [sprint]` | Generate PR checklist from pending tasks |
| `/premortem risk categories` | Return the 5 risk categories |

## Environment variables required

```
GRAPH_TENANT_ID        Microsoft Graph / Teams
GRAPH_CLIENT_ID
GRAPH_CLIENT_SECRET
TEAMS_MEETING_IDS      JSON: department → meeting ID
EXCEL_WORKBOOK_ID      SharePoint/OneDrive item ID
EXCEL_DRIVE_ID
GITHUB_TOKEN
ANTHROPIC_API_KEY
CLAUDE_MODEL           claude-sonnet-4-6
TEAMS_WEBHOOK_URL
```

## Plugin version

See `plugin.json` for current version, dependency graph, and file integrity checksums.
See `CHANGELOG.md` for full version history.
