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

## Testing Guide

### Tier 1 — No setup required (pure Claude Code, no backend)

These work immediately in any Claude Code session in this repo:

```
/standup departments
```
Lists all 10 supported departments and their Excel column schemas.

```
/standup standup devops Ahmed: we need to reduce memory on checkout-service. Sara: updating autoscaling policy for payment-api.
```
Claude extracts structured tasks from the transcript in real time.

```
/standup checklist devops 2026-06-16
```
Generates a GitHub PR checklist from pending DevOps tasks.

```
/standup generate standup_extractor
```
Writes the full `backend/ai/standup_extractor.py` FastAPI module.

```
/premortem risk categories
```
Returns the 5 DevOps risk categories with descriptions.

```
/premortem build order
```
Returns the 4-phase build sequence for the platform.

```
/premortem generate risk_predictor
```
Writes the full `backend/ai/risk_predictor.py` module.

```
/premortem explain layer 6
```
Explains the Teams Adaptive Card approval workflow in detail.

```
/premortem write prompt for standup extractor
```
Generates a production-ready Claude API system prompt + JSON schema.

---

### Tier 2 — GitHub Actions (demo mode, no secrets needed)

All 4 workflows run in demo mode when `PREMORTEM_BACKEND_URL` is not set.
They skip the backend call and exit green with a notice annotation.

1. Go to the repo → **Actions** tab
2. Select **PreMortem | Standup Sync**
3. Click **Run workflow** → choose a department → click **Run workflow**
4. The run completes green — job summary shows `STATUS=DEMO`

Repeat for **PreMortem | PR Analysis** via workflow_dispatch.

---

### Tier 3 — Full integration (backend deployed)

Add these secrets in **GitHub → Settings → Secrets and variables → Actions**:

| Secret | Purpose |
|---|---|
| `PREMORTEM_BACKEND_URL` | Base URL of the FastAPI backend |
| `PREMORTEM_API_TOKEN` | Bearer token for backend auth |
| `TEAMS_WEBHOOK_URL` | Teams incoming webhook URL |
| `ARGOCD_SERVER_URL` | ArgoCD server base URL |
| `ARGOCD_AUTH_TOKEN` | ArgoCD API token |

With secrets set, all 4 workflows call the live backend — transcripts are fetched from Teams, tasks are written to Excel, and approval cards are sent to Teams.

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
