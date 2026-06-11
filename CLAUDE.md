# PreMortem + StandupSync — Plugin Context

## What this plugin does

Transforms DEWA's daily standup meetings into a fully automated, AI-governed DevOps and business operations platform — split into two independent, composable skills.

```
Daily Standup (Teams) --> Transcript --> Claude --> Tasks --> Excel   [StandupSync]
        |
        v
Tasks become PR Checklists --> GitHub PR Gate                         [StandupSync]
        |
        v
PR Diff --> Risk Analysis --> Remediation --> Human Approval          [PreMortem]
        |
        v
ArgoCD Deploys --> Task Done in Excel --> Audit Log                   [PreMortem]
```

---

## Two Skills

### `/standup` — StandupSync
Standup intelligence layer. Extracts tasks from Teams transcripts, writes to Excel, injects PR checklists.

```
/standup departments
/standup standup [department] [transcript]
/standup checklist [department] [sprint]
/standup sheets sync [department]
/standup generate [module]
```

### `/premortem` — PreMortem
Predictive governance layer. Correlates PRs with standup tasks, predicts risk, routes fixes through human approval.

```
/premortem build order
/premortem risk categories
/premortem generate [module]
/premortem explain [layer]
/premortem write prompt for [module]
/premortem generate checksums
```

---

## Plugin Structure

```
premortem-standupsync/
├── CLAUDE.md                               ← you are here
├── OVERVIEW.md                             ← full documentation
├── CHANGELOG.md                            ← version history
├── plugin.json                             ← manifest: version, deps, checksums
├── .claude/
│   ├── settings.json                       ← hooks
│   ├── commands/
│   │   ├── premortem.md                    ← /premortem entry (governance)
│   │   └── standup.md                      ← /standup entry (intelligence)
│   └── skills/
│       ├── premortem/
│       │   └── SKILL.md                    ← PreMortem canonical skill
│       └── standup-sync/
│           └── SKILL.md                    ← StandupSync canonical skill
└── .github/
    └── workflows/
        ├── standup-checklist.yml           ← StandupSync: PR checklist visibility
        ├── standup-sync.yml                ← StandupSync: cron → transcript → Excel
        ├── pr-analyze.yml                  ← PreMortem: PR diff correlation
        └── remediation-deploy.yml          ← PreMortem: AI fix → ArgoCD gate
```

---

## Core Safety Principle

> **AI Suggests → Human Approves → GitOps Deploys**

---

## Environment Variables

```
GRAPH_TENANT_ID          GRAPH_CLIENT_ID          GRAPH_CLIENT_SECRET
TEAMS_MEETING_IDS        EXCEL_WORKBOOK_ID         EXCEL_DRIVE_ID
GITHUB_TOKEN             ANTHROPIC_API_KEY          CLAUDE_MODEL=claude-sonnet-4-6
TEAMS_WEBHOOK_URL        ARGOCD_SERVER_URL          ARGOCD_AUTH_TOKEN
```

See `plugin.json` for version, dependency graph, and file integrity checksums.
See `CHANGELOG.md` for full version history.
